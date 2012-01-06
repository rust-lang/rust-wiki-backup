These are guidelines for coding the Rust compiler and standard library.

## Predicates

The names of simple boolean predicates in std and core should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.