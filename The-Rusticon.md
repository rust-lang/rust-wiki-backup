This is currently a work in progress - feel free to alter or add definitions as necessary.

term | definition
-----|-----------
<a name="algebraic_data_type" /> algebraic data type | A type with a set of possible variants. These are declared in Rust using the `enum` keyword. More information can be found on [the Wikipedia article](http://en.wikipedia.org/wiki/Algebraic_data_type).
<a name="attribute" />attribute | A way of adding metadata to an item. Written as `#[attribute_name]` (other examples being `#[doc="Foo"]`, `#[doc(hidden="true")]`).
bikeshed | A [highly important discussion](http://www.catb.org/jargon/html/B/bikeshedding.html) about some non-fundamental part of the language (such as syntax or identifier names).
bors | The name of our continuous integration bot, a [Python script](https://github.com/graydon/bors) that checks for reviewed pull requests and runs the test on it, merging it if they pass.
box | (noun) An allocated chunk of memory, (verb) To place a value in such a box
built-in trait | A compiler-defined trait that is implicitly implemented for each eligible type. They are `Send`, `Share`, `Copy`, and `'static`.
cargo | The official package manager for Rust programs and libraries.
<a name="closure" />closure | Refers both to the type (`∣ ∣`, `proc()`) and the literal notation: `∣args∣ expression` (where expression can be a block, ie `∣x∣ { println(x.to_str()); 5}`). It is said to "close over" its environment; it can "capture" values from surrounding code. (*Note:* This is using a non-pipe unicode character because github doesn't like using that character in tables.)
crate | Rust's compilation unit, a single library or executable. Is the root of a namespace.
DST | [Dynamically sized type](http://smallcultfollowing.com/babysteps/blog/2014/01/05/dst-take-5/)
FFI | See _foreign function interface_.
foreign function interface | Calling code written in another language. Rust has a native C FFI, using `extern "C" fn`.
heap allocation | A dynamic allocation not on the stack.
HRTB | Higher-ranked trait bound. See [RFC 387](https://github.com/rust-lang/rfcs/blob/master/text/0387-higher-ranked-trait-bounds.md).
ICE | Internal compiler error: an internal assertion failure in the compiler, which always indicates a bug in the compiler. There should never be any user input that causes an ICE to happen, so if you see one, it's always a bug and reporting it is helpful (see [[HOWTO submit a RUST bug report]]).
inlining | Inlining is removal of a function call by including the function body directly into the callsite, enabling further optimizations. Controlled with the `inline` attribute: `#[inline(never)]`, `#[inline]` for a standard (though very strong) inline hint, and `#[inline(always)]`. Note that `#[inline]` is required for any cross-crate inlining.
Intermediate Representation | LLVM IR Code. It can be seen in text form by passing`--emit=ir` to rustc.
IR | See _Intermediate Representation_.
IRFY | Is Rust Fast Yet. Graphs tracking [how long the buildbots take to build + test](http://huonw.github.io/isrustfastyet/buildbot/). Also see its companion, [Is Rust Slim Yet](http://huonw.github.io/isrustfastyet/mem/).
lifetime | A piece of metadata that all references have. They represent the scope that the pointer is valid for. For more information, consult [the lifetime guide](http://static.rust-lang.org/doc/master/guide-lifetimes.html).
link-time optimization | A type of optimization performed by a compiler at link time. In rustlang, link-time optimization can only be performed on executables.
LTO | See _link-time optimization_. 
<a name="macro"/>macro | A type of syntax extension, defined with `macro_rules! name_of_macro`. Are a way of declaratively generating Rust from arbitrary tokens. For example, `assert!`, `debug!`, and `fail!` are macros. The standard macros can be found [in the documentation](http://static.rust-lang.org/doc/master/std/macros/index.html).
monomorphise | The act of generating specialized versions of generic constructs at compile time to improve run time performance. See [_Whole-Program Compilation in MLton_](http://mlton.org/References.attachments/060916-mlton.pdf) and [Niko Matsakis's answer on Stackoverflow](http://stackoverflow.com/a/14198060).
newtype struct | A [*tuple structure*](#tuple_structure) with a single unnamed field. For example `struct NodeIndex(int)`. Useful to create wrapper types.
owning pointer | `Box<T>`, a pointer to an owned box.
<a name="phantom_type" />phantom type | An `enum` with no variants. This cannot be constructed in safe code. See the [cheetsheet](https://github.com/mozilla/rust/wiki/Doc-FAQ-Cheatsheet#how-do-i-express-phantom-types) for an example.
plain old data (Pod) | Any value that can be safely copied by moving bits, including scalars, references, and structs containing only Pod. Types which are pod implement the `Copy` trait.
raw pointer | `*const T` or `*mut T`, a pointer to anything. Requires unsafe code to dereference, no static verification is done on them.
<a name="record_structure" />record structure | A struct declared with named fields, for example `struct Point { x: f32, y: f32 }`
<a name="reference" /> reference | A non-owning pointer to an object. Has an associated lifetime, to assert that what it points to is always valid data.
rust | <a name="rust" /> Rust is named after a [fungus](http://en.wikipedia.org/wiki/Rust_%28fungus%29) that is robust, distributed, and parallel. And, Graydon is a biology nerd. See [TL;DR](http://www.reddit.com/r/rust/comments/27jvdt/internet_archaeology_the_definitive_endall_source/)
rustdoc | The Rust documentation generator.
rustc | The Rust source code compiler.
sigil | (*Obsolete*) A character placed in front of a type, identifier or literal. In the context of Rust, this usually refered to the pointer symbols: `&`, `~`, `@`, and `*`.
stack allocation | All local variables are a stack allocation.
syntax extension | Code generation at compiletime, broken into three groups: declarative macros, which are described above as `macros`; procedural macros, which are written as Rust code that processes a token tree and produce an AST, and attributes.
task | Rust's fundamental unit of computation. Similar to a thread.
TLS | Task/Thread-Local Storage. See [these meeting minutes](https://github.com/rust-lang/meeting-minutes/blob/master/workweek-2014-08-18/libgreen-and-tls-redux.md)
trait | Rust's approach to ad-hoc polymorphism, and used for generics and dynamic dispatch. Also known as [type classes](http://en.wikipedia.org/wiki/Type_class).
<a name="tuple_structure" />tuple structure | A struct declared [without named fields](#record_structure), for example `struct Point(f32, f32)`
TWiR | This Week in Rust. cmr's [weekly summary](http://blog.octayn.net/) of Rust's development.
<a name="type_hint" />type hint | Syntax like `foo::<int>()` to give an explicit type for a generic function, method or struct constructor. Usually redundant due to type inference.
unbox | To get the value out of a box, usually destroying the box in the process.
<a name="unit_type" />unit type | The unit type, denoted `()`, permits only one value, also denoted `()`. Functions without return value have return type `()`. Sometimes called *nil*.
unit structure | A struct that only has one value, for example `struct Foo;` where `Foo` becomes the name of both the type and its only value. Works just the same as the *unit type* `()`, except it is a distinct type.
<a name="variant" />variant | One of the set of possible values that can be represented by an `enum`.
<a name="zero_variant_enum" />zero-variant enum | See [*phantom type*](#phantom_type).

## Syntaxicon

What is this syntax called?

*Note*: these names represent the current consensus on desired naming by #rust-internals and brson. Ask there before changing them.

syntax | name
-------|-----------
`()` | The [*unit type*](#unit_type).
`'static`, `'a` | A *lifetime*. May also be used to name a `loop` or a `for` clause (a label).
`Box<T>` | If `T` is a type, an *owning pointer* to `T`, and the pointer is to an *owned box*.
`&T`, `&'a T` | If `T` is a type, a [*reference*](#reference) to `T`, possibly with a *lifetime* (`'a`).
`∣ ∣`, `∣T∣ -> U` | A [*closure*](#closure). 
`&[T]`, `&'a [T]` | A *slice* with element type `T` and possibly with a *lifetime*.
`&mut [T]`, `&'a mut [T]` | A *mutable slice* with element type `T` and possibly with a *lifetime*. One can borrow a `&mut` to the elements of the slice.
`[T, ..n]` | An *array* of length `n`.
`[T]` | An *array* whose length is unknown (a dynamically-sized type).
`Box<Trait>:Send`, `&'a Trait:Send` | A *trait object* where `:Send` are the *kind bounds*.
`foo!()` | Either a [*macro*](#macro) or *syntax extension*.
`#[xyz]`, `#![xyz]` | An [*attribute*](#attribute).
`::<int>`, `: int` (in `let x: int ...`) | A [*type hint*](#type_hint). 

## Taking Arguments

AKA: "Help, where do I put the `mut`?"

There are a few combinations:

```rust
fn A(x: T) { }
fn B(x: &T) { }
fn C(x: &mut T) { }
fn D(mut x: T) { }
fn E(mut x: &T) { }
fn F(mut x: &mut T) { }
```

The `mut` before the `x` refers to the *binding*, whereas the `mut` after the `&` refers to *aliasability*. Only one `&mut` may exist to a given value at a given time, which means you are free to mutate it as you please. When a binding a mutable, you may reassign, so in `D` through `F`, in the body of the function you could write `x = foo();`. In `A` and `D`, the type is passed by-value, and if the type is "affine", it will be moved. The opposite of "affine" is "Copy", or "POD" (plain-old-data, jacked from C++), and these types will be copied rather than moved. That is, `A` and `D` would get a fresh copy that, if they mutated, the caller would not see the mutations. Additionally, `D` would be able to mutate `x` willy-nilly, as opposed to `A`, which would need a `let mut y = x;` or similar.

`B` and `E` take shared references. These are "aliasable" references, and multiple shared references can point at the same data at the same time. Any mutations that happen through a `&T` need to be safe in the face of this! Doing so is called "internal mutability", and is how the `RefCell` type works.
