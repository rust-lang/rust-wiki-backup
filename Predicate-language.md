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

TODO: details of the "safe" language and what's allowed