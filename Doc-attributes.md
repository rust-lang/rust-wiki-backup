# The many uses of attributes

## Meta items

Attributes are built from _meta items_, which have three forms:

* the simple _word_ meta item, which is just an identifier, as in `test`
* the _name-value_ meta item, an ident/literal pair, as in `target_os = "win32"`
* the _list_ meta item, a named list of additional meta items, as in `names(fred, barney, wilma)`

In _name-value_ items the literal may be any Rust literal type, though strings are seen most often.Meta items may nest arbitrarily: `args(yes, count = 10, names(fred, barny, wilma))`

Meta items occasionally show up outside of attributes, particularly in crate linkage specifications.

## Attributes

The primary use of meta items is for applying attributes to items, crates, methods, fields, and potentially other bits of code. Attributes provide additional information to the compiler and may also be accessed via reflection at runtime. An attribute is simply a meta item wrapped in attribute syntax - a preceding _pound_ sigil and enclosing _brackets_:

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

A crate's version is determined by the _link_ attribute, which is a list meta item containing metadata about the crate. This metadata can, in turn, be used in providing partial matching parameters to crate importing directives, denoted by the *extern mod* keyword.

All meta items within a link attribute contribute to the versioning of a crate, and two meta items, _name_ and _vers_, have special meaning and must be present in all crates compiled as shared libraries.

An example of a typical crate link attribute and linking to another crate:

    #[link(name = "std",
           vers = "0.1",
           uuid = "122bed0b-c19b-4b82-b0b7-7ae8aead7297",
           url = "http://rust-lang.org/src/std")];

    extern mod core(vers = "0.1");

## Build Configuration

Each time the compiler is run, it defines a set of meta items describing the compilation environment, called the _build configuration_. Several meta items are defined by default while others may be user-defined. The default items are:

* target_os - A name/value item with the value set to a string representing the target operating system, either "linux", "macos", or "win32".
* target_arch - A name/value item with the value set to a string representing the target architecture, currently always "x86".
* target_libc - A name/value item with the value set to a string naming the libc dynamic library.
* build_compiler - A name/value item with the value set to the name of the compiler.
* windows - A word item that is defined when building for Windows
* unix - A word item that is defined when building for Linux, OS X, or FreeBSD

## Conditional Compilation

The build configuration can be used to conditionally compile items, using the _cfg_ attribute. The cfg attribute is a list meta item containing a meta item that is compared to all meta items in the build configuration. If any match is found the item is compiled; if no match is found the item is parsed but not compiled.

An example of using conditional compilation to build different methods on different platforms:

    #[cfg(unix)]
    fn f() { ... }

    #[cfg(target_arch = "x86")]
    fn f() { ... }

## Unit testing

Rust includes a built-in [unit testing](Note unit testing) facility which makes use of attributes and conditional compilation.

## Other crate attributes

    // Tell the compiler to output a library by default
    #[crate_type = "lib"];

    // Don't link to core by default
    #[no_core];

Both of these are just informational, and no tool actually uses them yet.

    // A very short description of the crate
    #[comment = "The Rust core library"];
    #[license = "MIT"];

## Rustdoc

[Rustdocs](Doc using Rustdoc) are usually provided in documentation comments, but this is just sugar for doc attributes.

    /// Description
    fn oof() { }

    #[doc = "Description"]
    fn rab() { }

## Control compiler warnings

Compiler warnings and errors can be controlled to a very fine level with attributes.

    // I don't want to see any camel cased types
    #[deny(non_camel_case_types)]
    mod {
        type Foo = Bar;
        // Oh, but just this once
        #[allow(non_camel_case_types)]
        type baz = Foo;
        // No, I'm serious. I'm going to completely forbid those camel case
        // shenanigans so they can't be turned back on
        #[forbid(non_camel_case_types)]
        mod quux {
            ...
        }
    }

Also, `warn` works too. The complete list of lint warnings can be requested from `rustc` on the command line ... somehow.

## Foreign functions

    // Specify the calling convention used by foreign code
    #[abi = "cdecl"] extern { ... }
    #[abi = "stdcall"] extern { ... }

    // Don't try to link this library automatically
    #[nolink] extern mod foo { }

## Inlining

    #[inline] fn foo() { }
    #[inline(always)] fn bar() { }
    #[inline(never)] fn baz() { }

## Overriding the path to module files

    // Load our raygun module from flowerpot.rs instead of raygun.rs
    #[path = "flowerpot.rs"]
    mod raygun;

## Auto-serialization

The `auto_serialize` attribute can be used to automatically generate serializers for types.

## Relationship with macros
Please see [[Bikeshed syntax extension]]