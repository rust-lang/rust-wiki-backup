Here are some rough guidelines to Rust style. They are followed unevenly and there's not necessarily consensus about everything here. More work is required.

# Editor settings

  * Lines should not be longer than 100 characters.
  * Use spaces for indentation, not tabs.

# Naming conventions

## General

* Type names and enumeration variants should be in `CamelCase`.
* Functions, methods, and variables should be lowercase with underscores where it helps readability.
* Static variables should be in `ALL_CAPS`.
* Constructors are methods called `new` or `new_with_more_details`.
* When writing a binding to an external library, put the raw C bindings in a module called `ffi` (rather than `ll`). Do not create high-level bindings called `hl`.

## Trait naming

- trait examples: Copy, Owned, Const, Add, Sub, Num, Shr, Index, Encode, Decode, Reader, Writer, GenericPath
- extension traits: XUtil? When do you prefer default methods to extensions?
- avoid words with suffixes (able, etc). try to use transitive verbs, nouns, and then adjectives in that order

# Privacy

* Avoid (`pub impl Type { ... }`). Instead mark the individual methods public. This allows a reader to immediately tell which methods are public at a glance.

# Function declarations

Wrapped functions:

```
fn frobnicate(a: Bar,
              b: Bar)
              -> Bar {
    code;
}

fn foo<T:This,
       U:That>(
       a: Bar,
       b: Bar)
       -> Baz {
    code;
}
```

(Note: We need to adjust editors to do this automatically. This is not the current convention.)

The downside of this is, that the indendation level of many lines of code may change when the length of the function name changes.

# Platform-specific code

* When writing cross-platform code, try to group platform-specific code into a module called `platform`.
* Try to avoid `#[cfg]` directives outside this `platform` module.

# Imports

* Write `extern mod` directives first, then a blank line.
* Put local imports first, then external imports, then `pub use`.
* Avoid use of `use *`, except in tests.

Prefer to fully import types while module-qualifying functions, e.g.

```
use option::Option;
use cast;

let i: int = cast::transmute(Option(0));
```

# Namespacing

* It's OK to use `foo = bar;`

* Avoid importing functions, unless they're very common. Instead import up to the module you're going to use and then call `mod::func()`.

# Unit testing

  * Put tests in a test module at the bottom of the modules they test.
  * Use `#[cfg(test)]` to only compile when testing. Example:
```
#[cfg(test)]
mod test {
}
```

# Match expressions

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

# Comments

Prefer line comments and avoid block comments. Reason: it avoids the debate about whether to put stars on every line, etc.

In doc comments, write sentences that begin with capital letters and end in a period, even in the
short summary description.

Favor outer doc comments

```
/// Function documentation.
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

# Function definitions

TODO

# Crate/project naming

TODO


# Error messages and warnings

Rust code in error messages should be enclosed in backquotes.

Examples:

* ```found `true` in restricted position```

Error messages should use the pattern "expected \`X\`, found \`Y\`".

* ```mismatched types: expected `u16`, found `u8` ```

# Traits

[Trait](http://dl.rust-lang.org/doc/tutorial.html#traits) names should be capitalized and should follow the pattern of `Verb` or `Verber`, except in cases where no verb seems sensible.

Examples:

* ```Iterate```

# Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```

# Loops

A ```for``` loop is always preferable to a ```while``` loop unless the loop counts in a non-uniform way (making it difficult to express as a ```for```).

# Impls

Put `pub` modifiers on method names, not on the impls.


# Questions and TODO

* Do we want to prefer 'top-down' organization?
* Prefer unsafe pointers to unsafely transmuting borrowed pointers