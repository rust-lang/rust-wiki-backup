# Under construction -- don't take this page seriously yet!

## Preliminaries
Typestate constraints are _predicates_: applications of a Rust function to one or more arguments. So a predicate has the form:

    check(p(x, y, z));

where `p` must be defined as a known function, and `x` (and so on) must be names of local slots or literals. The part inside the `check` is the constraint.

## The problem: specifying the predicate language
The question is, what is `p` allowed to be? In the current design, `p` must be declared with the keyword `pred` instead of `fn`. Items declared with `pred` must fall within a very conservative approximation of the set of observably-pure (referentially transparent) functions. For example, `pred` functions can't use assignment at all, and can only call other `pred` functions.

The current design may limit expressivity overly. Consider the following hypothetical function declarations

    fn substr(uint start, uint end, str s) : le(start, end), le(end, str::len(s)) -> str {
       ...
    }

    pred le(uint i, uint j) -> bool { i <= j }

The definition of the standard library function `str::len` might look something like:

    fn len(str s) -> uint {
       let uint l = 0u;
       for (char c in s) {
          l += 1u;
       }
       l
    }

The code as it stands above wouldn't be accepted by the current version of `rustc`, for several reasons. One is that predicate arguments can't be calls, so `str::len(s)` actually can't appear as an argument to `le`. We could solve the problem by writing a predicate relating a `uint` with a string:

    pred le_length_of(uint i, str s) -> bool {
       auto l = str::len(s);
       i <= l
    }

However, the compiler wouldn't accept this code either, because a pred can't call a `fn` -- `str::len` -- and we can't rewrite `str::len` as a pred straightforwardly, given its use of assignment. It's possible to imagine writing a "pred" version of `str::len` that computes the string's length with tail recursion rather than a loop, but this solution raises two distinct, and major, issues: first, it's undesirable from a code reuse point of view; second, we might like to use more complicated functions that `str::len` in predicates, where it would be harder to write a pure version.

##Solutions

So far, three major solutions have been proposed:

1. Dan: Eliminate the distinction between `fn` and `pred` entirely, and allow any function to be the operator in a typestate constraint.
2. Graydon: "preds can call non-pred functions, but can still only apply to immutable values. With further restriction: can only apply to immutable transparent values -- nothing containing objects, functions (or channels/ports/tasks, which will probably all be in libraries as obj and fn types anyways)." (From comments on Issue #693)
3. Dave/Tim: Compromise solution where we specify a "safe" predicate language and guarantee that the compiler will reject any programs that attempt to call constrained functions in contexts where those functions' constraints may not be satisfied, as long as all predicates are implemented in the "safe" subset; in addition, we provide an unsafe predicate language where guarantees are weaker, but the programmer can write more powerful predicates.

##Problems with solutions

###Solution 1

Solution 1 may have surprising results. Consider:

    fn f(int x) : p(x) -> bool { x == 5 && is_it_raining() }

    check p(x);
    foo();
    f(x);

where `is_it_raining()` queries some mutable state (perhaps over the network) and the call to `foo` may also mutate that state. Then, the predicate `p(x)` may not hold just before the call to `f`, even though it's true after the `check`.

Effectively, the meaning of a predicate can now depend on implicit state, and thus the restriction that predicate arguments must be immutable is no longer useful either. Thus, it becomes somewhat unclear what guarantee the compiler affords about the meanings of predicates.

The argument for this solution is that the compiler never tried to guarantee that there was a relationship between the implementation of a predicate `p` with type `int -> bool` and the semantics of the refinement type `{x:int | p(x)}` -- this is always a proof obligation on the user. So the nature of the guarantee is unclear in the first place.

###Solution 2

1. #### Wrappers
    Solution 2 still allows predicates to call any function: the user just has to write a `pred` wrapper around it. For example:

        pred p(int x) -> bool { x == 5 && is_it_raining() }
        fn is_it_raining() { /* does some network communication with a hard-wired server name */ }

    This code obeys the rules proposed in solution 2, because it calls `is_it_raining()` with only immutable arguments (that is, no arguments). But `is_it_raining` may then call any function, including functions that interact with mutable state.

1. #### Predictability

    Dave points out: "if the integration of the predicate checking system with the control flow analysis doesn't actually provide any guarantees about the result of the predicate, then if programmers started to discover subtle bugs due to control-sensitivity, they'd lose trust in the predicate type system." For example, if the predicate type system is trustworthy, that means that the programmer should be able to expect that this program will never `fail` at runtime:

        pred p(int x) -> bool { /* maybe calls non-referentially-transparent functions */ }
        fn f(int y) : p(y) -> int {
           check p(y);
           ...
        }
        fn main() {
           let x = 5; 
           check p(x);
           /* sleep for 8 hours, yielding to other tasks */
           log f(x);
        }

    If the typestate analysis is sound with respect to any reasonable semantics, the user should expect that the `check` inside `f` should always succeed, as the compiler is supposed to prevent `f` from being called on a `y` for which `p` is false. But since `p` here has non-referentially-transparent semantics, its meaning is actually dependent on some hidden state, not just on `y`. I think this behavior could surprise the user, and cause them to distrust the predicate type system (making them not want to rely on it, and stop using it).

1. #### Security

    If it's easy to silence typestate errors by wrapping a truly impure predicate with a trivial `pred` wrapper, that raises the temptation for users to do so in order to shut the compiler up, without understanding that this obliges them to prove that it is safe to do so, and without understanding the risks they thus introduce. This temptation could be addressed with a compiler warning (like: "Warning: `pred` function calls impure predicates"), but that runs the risk of overloading the programmer with too many warnings and causing them to ignore warnings altogether. The warning-based solution also doesn't force the programmer to specify clearly what properties they rely on the compiler checking and which they need to prove externally. 

    A possible response is that the user is responsible for understanding how the typestate system works well enough to be able to predict when their code may rely on an invariant that the compiler isn't actually checking. However, typestate is already going to be new to most users, and we don't want to introduce the additional cognitive load of making programmers think through tricky corner cases in a system that they probably won't find intuitive. Instead, we would rather design the language so as to force the programmer to declare their intentions. With the right syntax, we can instead make it unlikely that the programmer will accidentally write code that entails a proof obligation they aren't aware of. We would prefer to make the boundaries of the statically-safe language as clear as possible, as well as requiring the user to make it clear in their code that they know when they've stepped outside those boundaries. Wrapping an unsafe function in a predicate would violate that principle, as it's unclear in that case whether the programmer intended to write unsafe code.

1. #### Mutable data

    Conceivably, it might actually be useful to write predicates that accept mutable data structures. For example, we might want to check an invariant on a mutable data structure. It would be good to document that the user must not mutate the data structure between checks, and calls that rely on those checks. Solution 2 forbids such a scenario completely by disallowing any predicate applications with mutable arguments.

    As an example, consider a balanced binary tree type that allows destructive updates. If the user wants to enforce statically that certain operations take a balanced tree as an argument, they could write:
         
        fn insert(t: &Tree) : is_balanced(t) { ... }

    We would expect that there are intermediate states (like during a rebalancing operation) where ```is_balanced(t)``` would not hold for a given tree ```t```. Solution 2 would disallow this example, since it disallows predicates that take mutable types, like ```Tree```. Yet it seems like a fairly common scenario to have invariants on a mutable data structure that may or may not hold at an arbitrary program point, but need to hold at certain program points for correctness.

##Pure/impure language solution

This proposal would add a `check-volatile` keyword to the language (name subject to change), in addition to the existing `check` keyword. The difference between `check(p(x, y, z))` and `check-volatile(q(x, y, z))`, `q` could be an arbitrary function while `p` would have to be declared with `pred` (similarly to the current compiler) and its body would be checked according to a set of effect-checking rules. 

The basic idea is that the compiler guarantees that the invariants declared as part of a function precondition will actually be true at all of its call sites, _if_ all typestate predicates are referentially transparent. The problem is that a predicate may be semantically referentially transparent (like `le_length_of`), but might fail a simple syntactic test for referential transparency (lack of assignment expressions).

We introduce a distinction between pure and impure (general) predicates so that when a programmer knows that a predicate is referentially transparent but can't prove it to the compiler, they can still use it anyway, in a `check-volatile` expression. They then incur a proof obligation that the predicate really is referentially transparent; the compiler guarantees nothing in this case.

In practice, we would expect that most predicates will be referentially transparent (and obviously so, at that), but cases like the `str::len` example suggest we don't want to limit expressivity unduly or force code duplication.

We actually distinguish between three sorts of functions:

* **General** functions are declared with `fn`, and subject to no restrictions beyond Rust's usual type system.
* **Declared-pure** functions are declared with `pure-fn`, and are subject to the effect checking rules described in what follows. A declared-pure function may have any return type except for `()` or `_|_`, which would be nonsensical. A declared-pure function that has a boolean return type -- called a "predicate", but not syntactically distinct from other declared-pure functions -- may appear in a typestate constraint. A predicate may call declared-pure functions that have non-boolean return types.
* **Pure** functions are declared with `fn`, and subject to the effect checking rules as well. The motivation is that the compiler is free to "promote" some general functions to pure status (to infer purity for them) if it can infer that the function's observable behavior is that of a pure function. A pure function that has a boolean return type is also a predicate.

Every declared-pure function is pure, as enforced by the effect checking rules. For modularity purposes, the compiler will only promote a `fn` function to pure status if it's within the same crate as the caller. The reason is that otherwise, a change to the implementation of a pure function in one crate could cause code in another crate to fail to compile, which would be surprising.

To summarize, `check` takes a declared-pure or pure predicate, while `check-volatile` takes any function with a boolean return type.

A predicate must have a known definition: in `check(p(...))`, `p` may not be bound to a function argument.

The arguments to a declared-pure function must be immutable and transparent. A general function whose arguments may be mutable or opaque can't be promoted to pure status.

### Effect-checking rules

A declared-pure function may not

* call general functions (it may call declared-pure functions, and other pure functions), except inside the antecedent of an `if check-volatile` expression (see next section)
* move, assign or swap to anything other than a local slot
* receive on a channel (sending is OK)
* refer to upvars

These rules also specify the conditions under which the compiler will promote a general function to pure status.

These rules may appear similar to the effect system (impure/pure functions) in earlier versions of the language, and they are. However, a major difference is the ability to opt out of the effect-checking rules by using general predicates. Impurity is the default for _declaring_ functions, rather than purity, while purity is the default for _checking_ predicates. We also foreclose complicated issues such as effect polymorphism by simplifying the system. Finally, we do a limited form of effect masking (pure functions may modify local state).

### Dangerous calls

We extend the principle of allowing the user to violate safety rules as long as they declare their intentions to also allow calls from declared-pure functions to general functions. In addition to the `check-volatile` expression, we introduce an `if check-volatile` expression form, which works the way `if check` does except that (as in a `check-volatile` expression), the operator in the constraint may be a general function. For example:

    pred f(int x) -> bool {
      if check-volatile(other_crate::fn_that_really_acts_pure()) {
         ...
      }
      else {
         ...
      }
    }

## Summary
Allowing general (possibly-impure) predicates has no effect on type soundness; only on the guarantee to the user about how much confidence they can have about the relationship between the high-level invariants in the code they write, and in the code they run. Declaring uses of general predicates as unsafe (using the `check-volatile` keyword) should be a warning sign to the user that they should tread carefully (that is, the predicate writer and the predicate user have a shared proof obligation to ensure that semantically, predicates and their uses are referentially transparent). At the same time, the distinction between general and pure predicates affords the expressivity to use any Rust function as a predicate.

In the best case, the predicate writer ensures that the predicate is actually referentially transparent in all cases, which means the predicate user's obligations are vacuous. In general, we would expect that there is a set of certain restricted conditions under which the predicate behaves referentially transparently, in which case it's the predicate writer's job to specify those conditions and the predicate user's job to satisfy them at all call sites.
