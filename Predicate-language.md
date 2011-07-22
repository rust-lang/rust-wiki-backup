Typestate constraints are _predicates_: applications of a Rust function to one or more arguments. So a predicate has the form:

    check(p(x, y, z));

where `p` must be defined as a known function, and `x` (and so on) must be names of local slots or literals. The part inside the `check` is the constraint.

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

However, the compiler wouldn't accept this code either.