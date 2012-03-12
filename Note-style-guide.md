These are coding guidelines for the Rust compiler and standard library.

## Constructor functions

If possible then constructor functions should simply be named for the type that they construct, e.g. `str_reader`. When disambiguation is needed then they should be prefixed with `new_`. One common exception is functions that create values be converting from other values, in which case they should have a `from_` prefix, like `vec::from_elem`.

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