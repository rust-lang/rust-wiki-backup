These are coding guidelines for the Rust compiler and standard library.

## Constructor functions

If possible then constructor functions should simply be named for the type that they construct, e.g. `StrReader`. When disambiguation is needed then they should be prefixed with `new_`, and written in lower case like, `new_str_reader_with_foo`. One common exception is functions that create values by converting from other values, in which case they should have a `from_` prefix, like `vec::from_elem`.

Examples:

* `io::StrReader`
* `task::new_task_builder`
* `vec::from_elem`

## Error messages and warnings

Rust code in error messages should be enclosed in backquotes.

Examples:

* ```found `true` in restricted position```

Error messages should use the pattern "expected \`X\`, found \`Y\`".

* ```mismatched types: expected `u16`, found `u8` ```

## Traits

[Trait](http://dl.rust-lang.org/doc/tutorial.html#traits) names should be capitalized and should follow the pattern of `Verb` or `Verber`, except in cases where no verb seems sensible.

Examples:

* ```Iterate```

## Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```

## Loops

A ```for``` loop is always preferable to a ```while``` loop unless the loop counts in a non-uniform way (making it difficult to express as a ```for```).