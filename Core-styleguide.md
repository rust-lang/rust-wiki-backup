These are coding guidelines for the Rust compiler and standard library.

## Naming

### Interfaces

Interface names should be verbs.

Examples:

* ```iterate```

### Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```