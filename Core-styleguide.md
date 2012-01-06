These are guidelines for writing Rust compiler and standard library code.

## Naming

### Interfaces

Interface names should be verbs.

### Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```