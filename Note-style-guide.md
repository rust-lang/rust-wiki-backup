These are coding guidelines for Rust code.

## Naming conventions

- 100 column lines, 4 space indent, no tabs
- Type names, variants - camel case
- function - lowercase with underscores where it helps readability
- Static variables are capitalized

on type implementations, make default private, add pub explicitly where wanted

Trait naming?

- trait examples: Copy, Owned, Const, Add, Sub, Num, Shr, Index, Encode, Decode, Reader, Writer, GenericPath
- extension methods: XUtil? Prefer default methods
- avoid things with prefixes (able, etc). try to use transitive verbs, nouns, and then adjectives in that order



Constructors are static methods called `new` or `new_with_more_details`.

use ffi instead of _hl or ll or raw as the module for holding the extern definitions

## function declarations

Wrapped functions:

```
fn frobnicate(a: bar,
              b: bar)
              -> bar {
    code;
}
```

note: need to adjust editors to do this automatically - this is not the current convention

# cfg

avoid using cfg directives outside of platform namespace

# imports

extern mods, blank line
local imports first, then external imports, then pub use.

avoid use of *, except in tests

# namespacing

ok to use use `foo = bar;`

avoid import of functions, instead `mod::func()`

# editor settings

spaces not tabs
line length 100

# testing

put tests in a test module at the bottom of the modules they test. Use `#[cfg(test)]` to only compile when testing.

```
#[cfg(test)]
mod test {
}
```

# match expressions

deref the match target if you can. prefer 

```
match *foo {
    X(...) => ...
    Y(...) => ...
}
```

instead of

```
match foo {
    @X(...) => ...
    @Y(...) => ...
}
```

multiple patterns in a single arm:

```
match  foo {
    bar(*)
    | baz => quux,
    x
    | y
    | z => {
        quuux
    }
}
```

only omit braces for single expressions

```
match foo {
    bar => baz,
    quux => {
        do_something();
        do_something_else();
    }
}
```

# comments

Prefer line comments and avoid block comments. Reason: it avoids the debate about whether to put stars on every line, etc.

Favor outer doc comments

```
/// Function documentation
fn foo() {
    ...
}
```

Only use inner doc comments to document crates and file-level modules.

```
//! The core library.
//!
//! The core library is a something something...
```

Use full sentences that start with capitals and end with a period. See [[Doc-using-rustdoc]].

# Module organization

Put types first, then implementations

Do we want to prefer 'top-down' organization?

# Function definitions

TODO

# Crate/project naming

TODO


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


