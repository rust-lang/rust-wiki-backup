Here are some rough guidelines to Rust style. They are followed unevenly and there's not necessarily consensus about everything here. More work is required.

# Editor settings

  * Lines should not be longer than 100 characters.
  * Use spaces for indentation, not tabs.

# Naming conventions

## General

* Type names and enumeration variants should be in `CamelCase`.
* Acronyms should be camel case, too: `Uuid`, not `UUID`.
* Functions, methods, and variables should be `lowercase_with_underscores` where it helps readability.
* Static variables should be in `ALL_CAPS`.
* Constructors are methods called `new` or `new_with_more_details`.
* Constructors that simply convert from another type are methods called `from_foo`.
* When writing a binding to an external library, put the raw C bindings in a module called `ffi` (rather than `ll`). Do not create high-level bindings called `hl`.
* lifetime names are lowercase and are often simply 'a

## Trait naming

- trait examples: Copy, Owned, Const, Add, Sub, Num, Shr, Index, Encode, Decode, Reader, Writer, GenericPath
- extension traits: FooUtil. However, prefer default methods to extension traits.
- avoid words with suffixes (able, etc). try to use transitive verbs, nouns, and then adjectives in that order

## Converting between types
Functions for converting between types should attempt to follow this:

- `as`: cheap conversions that are normally just converting a reference to a different type, but not changing the in-memory representation, e.g. `string.as_bytes()` gives a `&[u8]` view into a `&str`. These don't consume the convertee.
- `to`: expensive conversions that may allocate and copy data, e.g. `string.to_owned()` copies a `&str` to a new `~str`. These also don't consume the convertee.
- `into`: conversions that consume the convertee and almost always don't allocate new memory (they may change the in-memory representation, but will normally do so in-place), these are cheaper than `to` conversions, e.g. `string.into_bytes()` converts a `~str` to a `~[u8]` without copying.

These are not hard rules, since generically implemented functions mean that an `into` conversion that doesn't copy/allocate is impossible for some types, e.g. `string.into_owned()` (from the `Str` trait) doesn't copy when `string` is `~str`, but it is forced to when `string` is `&str` or `@str`.

## Iterators

Naming iterators are often a little tricky because their names can get quite verbose quite quickly. Since #11001 landed, we've standardized on iterator naming conventions. When naming an iterator, follow these rules from top-to bottom (short circuiting when you hit a relevant one).

1. If the iterator is yielding something that can be described with a noun, the iterator should be called the pluralization of the noun (an iterator yielding words is called Words)
2. An iterator over the members of a container should have the base name of `Items`, with different flavors deriving from this name. Different flavors are applied in order of top-to-bottom in this list
  * Moving iterators have the prefix of `Move`
  * If the default iterator yields an immutable reference, an iterator yielding a mutable reference has the prefix `Mut`
  * Reverse iterators have the prefix of `Rev`
3. If these rules would result in a name that might cause confusion, pick a less confusing name.

These rules are a little vague, and that's partly on purpose. The general idea is to be concise and consistent. Examples through libstd and libextra should showcase how we expect iterators to be named.

# Function declarations

```
fn frobnicate(a: Bar, b: Bar) -> Bar {
    // code;
}

fn foo<T:This, U:That>(a: Bar, b: Bar) -> Baz {
    // code;
}
```

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
    bar(*) | baz => quux,
    x | y | z => {
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

[Trait](http://doc.rust-lang.org/doc/master/tutorial.html#traits) names should be capitalized and should follow the pattern of `Verb` or `Verber`, except in cases where no verb seems sensible.

Examples:

* ```Iterate```

# Impls

* Avoid `pub impl Type { ... }`. Instead put `pub ` modifiers on the method names. This allows a reader to immediately tell which methods are public at a glance.

# Predicates

The names of simple boolean predicates should start with "is_" or similarly be expressed using a "small question word".

The notable exception are generally established predicate names like "lt", "ge", etc.

Examples:

* ```is_not_empty```

# Loops

A ```for``` loop is always preferable to a ```while``` loop unless the loop counts in a non-uniform way (making it difficult to express as a ```for```).

# Questions and TODO

* Do we want to prefer 'top-down' organization?
* Prefer unsafe pointers to unsafely transmuting borrowed pointers
