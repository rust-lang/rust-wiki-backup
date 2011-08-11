# Predicate language

## Preliminaries
Typestate constraints are _predicates_: applications of a Rust function to one or more arguments. So a predicate has the form:

    check(p(x, y, z));

where `p` must be defined as a known function, and `x` (and so on) must be names of local slots or literals. The part inside the `check` is the constraint.