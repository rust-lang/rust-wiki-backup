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

The problem with solution 1 is that it may have surprising results. Consider:

    fn f(int x) : p(x) -> bool { x == 5 && is_it_raining() }

    check p(x);
    foo();
    f(x);

where `is_it_raining()` queries some mutable state (perhaps over the network) and the call to `foo` may also mutate that state. Then, the predicate `p(x)` may not hold just before the call to `f`, even though it's true after the `check`.

Effectively, the meaning of a predicate can now depend on implicit state, and thus the restriction that predicate arguments must be immutable is no longer useful either. Thus, it becomes somewhat unclear what guarantee the compiler affords about the meanings of predicates.

The argument for this solution is that the compiler never tried to guarantee that there was a relationship between the implementation of a predicate `p` with type `int -> bool` and the semantics of the refinement type `{x:int | p(x)}` -- this is always a proof obligation on the user. So the nature of the guarantee is unclear in the first place.

The problem with solution 2 is that it still allows predicates to call any function: the user just has to write a `pred` wrapper around it. For example:

    pred p(int x) -> bool { x == 5 && is_it_raining() }
    fn is_it_raining() { /* does some network communication with a hard-wired server name */ }

This code obeys the rules proposed in solution 2, because it calls `is_it_raining()` with only immutable arguments (that is, no arguments). But `is_it_raining` may then call any function, including functions that interact with mutable state.

##Pure/impure language solution

This proposal would add a `check-volatile` keyword to the language (name subject to change), in addition to the existing `check` keyword. The difference between `check(p(x, y, z))` and `check-volatile(q(x, y, z))`, `q` could be an arbitrary function while `p` would have to be declared with `pred` (similarly to the current compiler) and its body would be checked according to a set of effect-checking rules. 

The basic idea is that the compiler guarantees that the invariants declared as part of a function precondition will actually be true at all of its call sites, _if_ all typestate predicates are referentially transparent. The problem is that a predicate may be semantically referentially transparent (like `le_length_of`), but might fail a simple syntactic test for referential transparency (lack of assignment expressions).

We introduce a distinction between pure and impure (general) predicates so that when a programmer knows that a predicate is referentially transparent but can't prove it to the compiler, they can still use it anyway, in a `check-volatile` expression. They then incur a proof obligation that the predicate really is referentially transparent; the compiler guarantees nothing in this case.

In practice, we would expect that most predicates will be referentially transparent (and obviously so, at that), but cases like the `str::len` example suggest we don't want to limit expressivity unduly or force code duplication.

We actually distinguish between three sorts of functions:

* **General** functions are declared with `fn`, and subject to no restrictions beyond Rust's usual type system.
* **Declared-pure** functions are declared with `pred`, and subject to the effect checking rules described in what follows.
* **Pure** functions are declared with `fn`, and subject to the effect checking rules as well. The motivation is that the compiler is free to "promote" some general functions to pure status if it can infer that the function's observable behavior is that of a pure function.

Every declared-pure function is pure, as enforced by the effect checking rules. For modularity purposes, a `fn` function will only be treated as pure if it's within the same crate as the caller. The reason is that otherwise, a change to the implementation of a pure function in one crate could cause code in another crate to fail to compile, which would be surprising.

To summarize, `check` takes a declared-pure or pure predicate, while `check-volatile` takes any function with a boolean return type.

A pure function must have a known definition: in `check(p(...))`, `p` may not be bound to a function argument.

The arguments to a pure function must be immutable and transparent.

### Effect-checking rules

A pure function may not

* call general functions (it may call declared-pure functions, and other pure functions)
* move, assign or swap to anything other than a local slot
* receive on a channel (sending is OK)
* refer to upvars

These rules may appear similar to the effect system (impure/pure functions) in earlier versions of the language, and they are. However, a major difference is the ability to opt out of the effect-checking rules by using general predicates. Impurity is the default for _declaring_ functions, rather than purity, while purity is the default for _checking_ predicates. We also foreclose complicated issues such as effect polymorphism by simplifying the system. Finally, we do a limited form of effect masking (pure functions may modify local state).

## Summary
Allowing general (possibly-impure) predicates has no effect on type soundness; only on the guarantee to the user about how much confidence they can have about the relationship between the high-level invariants in the code they write, and in the code they run. Declaring uses of general predicates as unsafe (using the `check-volatile` keyword) should be a warning sign to the user that they should tread carefully (that is, that they have a proof obligation to ensure that semantically, their predicates are referentially transparent). At the same time, the distinction between general and pure predicates affords the expressivity to use any Rust function as a predicate.