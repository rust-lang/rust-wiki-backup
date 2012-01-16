These are coding guidelines for the Rust compiler and standard library.

## Error messages and warnings

Rust code in error messages should be enclosed in backquotes.

Examples:

* ```found `int` but expected `str` ```

## Interfaces

[Interface](Note Interfaces) names should be verbs.

Examples:

* ```iterate```

## Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```