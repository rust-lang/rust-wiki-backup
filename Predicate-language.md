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

##Safe/unsafe language solution

**not finished yet**

(philosophy: if you declare your stuff to be referentially transparent, we'll help you. if not, we won't stop you from expressing your invariants, but all bets are off. in practice we would think most people want to write referentially transparent things, but cases like the `str::len` example above show we don't want to rule 
out useful things or make people duplicate code.)

Pure = declared with pred. Safe = not declared with pred, but the compiler infers that it *could* have been. (For modularity, allow people to use a safe function like a pure function, but within the same module (crate?) only.)

This proposal would add a `check-referentially-opaque` keyword to the language (name subject to change), in addition to the existing `check` keyword. Comparing `check(p(x, y, z))` with `check-referentially-opaque(q(x, y, z))`, `q` could be an arbitrary function while `p` would have to be declared with `pred` (similarly to the current compiler) and its body would be checked according to the following set of rules. 

A safe function must have a known definition: in `check(p(...))`, `p` may not be bound to a function argument.

The arguments to a safe function must be immutable and transparent.

A safe function may *not*:
* call functions declared with `fn`
* move, assign or swap to anything other than a local slot
* receive on a channel (sending is OK)
* refer to upvars

(This sounds similar to the effect system... and it is... but there's the ability to opt out (major difference)... and it avoids the really complicated issues like effect polymorphism / higher-orderness... and we would do effect masking.)

(Discovery / change to implementation breaking safety?)

Summary: Can do anything in the pred language, but if you want to do something not obviously safe you have to communicate your intention, reminding yourself that you have a proof obligation to ensure that your predicates _behave_ in a way that's referentially transparent, even if they're too complicated for the compiler to prove their referential transparency.