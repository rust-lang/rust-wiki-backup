# Notes on attributes

## Meta items

Several places in the Rust language allow the specification of metadata about code, called _meta items_, primarily in relation to _attributes_. Meta items have three forms: the simple _word_ meta item, which is just an identifier; the _name-value_ meta item, an ident/literal pair; and the _list_ meta item, a named list of additional meta items.

Examples:

    // a word meta item, must be a valid identifier
    test

    // name/value meta items
    target_os = "win32"
    index = 10

    // list meta items
    cfg(target_os = "win32")
    names(fred, barney, wilma)

## Attributes

The primary use of meta items is for applying attributes to items and crates. Attributes provide additional information to the compiler and may also be accessed via reflection at runtime. An attribute is simply a meta item wrapped in attribute syntax - a preceding _pound_ sigil and enclosing _brackets_:

    #[test]

or

    #[cfg(target_os = "linux")]

Any number of items may precede the definition of an item (or native item):

    #[test]
    #[cfg(target_os = "linux")]
    mod m { ... }

Additionally, attributes may be applied by placing them immediately inside the openining brace of certain items, and terminating each with a semi-colon:

    mod m {
        #[test];
        ...
    }

Similarly, to apply attributes to a crate, they should be specified at the beginning of a crate file, each terminated with a semicolon.

## Crate Linkage Attributes

A crate's version is determined by the _link_ attribute, which is a list meta item containing metadata about the crate. This metadata can, in turn, be used in providing partial matching parameters to syntax extension loading and crate importing directives, denoted by the *syntax* and *use* keywords respectively.

All meta items within a link attribute contribute to the versioning of a crate, and two meta items, _name_ and _vers_, have special meaning and must be present in all crates compiled as shared libraries.

An example of a typical crate link attribute:

    #[link(name = "std",
           vers = "0.1",
           uuid = "122bed0b-c19b-4b82-b0b7-7ae8aead7297",
           url = "http://rust-lang.org/src/std")];

## Build Configuration

Each time the compiler is run, it defines a set of meta items describing the compilation environment, called the build configuration. Several meta items are defined by default while others may be user-defined. The default items are:

* target_os - A name/value item with the value set to a string representing the target operating system, either "linux", "macos", or "win32".
* target_arch - A name/value item with the value set to a string representing the target architecture, currently always "x86".
* target_libc - A name/value item with the value set to a string naming the libc dynamic library.
* build_compiler - A name/value item with the value set to the name of the compiler.

## Conditional Compilation

The build configuration can be used to conditionally compile items, using the _cfg_ attribute. The cfg attribute is a list meta item containing a meta item that is compared to all meta items in the build configuration. If any match is found the item is compiled; if no match is found the item is parsed but not compiled.

An example of using conditional compilation to build different methods on different platforms:

    #[cfg(target_os = "linux")]
    fn f() { ... }

    #[cfg(target_os = "macos")]
    fn f() { ... }

## Unit testing

Rust includes a built-in [[unit testing]] facility which makes use of attributes and conditional compilation.

## Relationship with macros
Please cf [[Syntax-extension]]