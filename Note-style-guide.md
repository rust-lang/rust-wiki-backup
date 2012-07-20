These are coding guidelines for the Rust compiler and standard library.

## Constructor functions

If possible then constructor functions should simply be named for the type that they construct, e.g. `str_reader`. When disambiguation is needed then they should be prefixed with `new_`. One common exception is functions that create values by converting from other values, in which case they should have a `from_` prefix, like `vec::from_elem`.

Examples:

* `io::str_reader`
* `task::new_task_builder`
* `vec::from_elem`

## Error messages and warnings

Rust code in error messages should be enclosed in backquotes.

Examples:

* ```found `true` in restricted position```

Error messages should use the word "allowed", rather than synonyms like "permitted" or antonyms like "prohibited".

Examples:

* ```type parameters are not allowed on this type```

Error messages should use the pattern "expected \`X\`, found \`Y\`".  (Note: right now, we're inconsistent about this; most of the compiler uses "expected \`X\` but found \`Y\`".  Still, we consistently use the backquotes and the words "expected" and "found", rather than, say, "expecting" and "got".

* ```mismatched types: expected `u16`, found `u8` ```

## Interfaces

[Interface](Note Interfaces) names should be verbs.

Examples:

* ```iterate```

## Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```