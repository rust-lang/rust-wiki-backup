This is currently a work in progress - feel free to alter or add definitions as necessary.

term | definition
-----|-----------
<a name="algebraic_data_type" /> algebraic data type | A type with a set of possible variants. These are declared in Rust using the `enum` keyword. More information can be found on [the Wikipedia article](http://en.wikipedia.org/wiki/Algebraic_data_type).
<a name="attribute" />attribute | A way of adding metadata to an item. Written as `#[attribute_name]` (other examples being `#[doc="Foo"]`, `#[doc(hidden="true")]`).
bikeshed | A [highly important discussion](http://www.catb.org/jargon/html/B/bikeshedding.html) about some non-fundamental part of the language (such as syntax or identifier names).
<a name="borrowed_pointer" /> borrowed pointer | Also known as a "reference". It references an object without taking ownership of it. Has an associated lifetime, to assert that it is always valid.
bors | The name of our continuous integration bot, a [Python script](https://github.com/graydon/bors) that checks for reviewed pull requests and runs the test on it, merging it if they pass.
box | An allocated chunk of memory.
built-in trait | A compiler-defined trait that is implicitly implemented for each eligible type. They are `Send`, `Freeze` and `'static`.
<a name="closure" />closure | Refers both to the type (`&fn`, `~fn`) and the literal notation: `∣args∣ expression` (where expression can be a block, ie `∣x∣ { println(x.to_str()); 5}`). It is said to "close over" its environment; it can "capture" values from surrounding code. (*Note:* This is using a non-pipe unicode character because github doesn't like using that character in tables.)
crate | Rust's compilation unit, a single library or executable. Is the root of a namespace.
FFI | See _foreign function interface_.
foreign function interface | Calling code written in another language. Rust has a native C FFI, using `extern "C" fn`.
heap allocation | A dynamic allocation performed either by `~` or `@`, which call to `malloc` in the default runtime (which is a statically-linked `jemalloc`).
ICE | Internal compiler error: an internal assertion failure in the compiler, which always indicates a bug in the compiler. There should never be any user input that causes an ICE to happen, so if you see one, it's always a bug and reporting it is helpful (see [[HOWTO submit a RUST bug report]]).
inlining | Inlining is removal of a function call by including the function body directly into the callsite, enabling further optimizations. Controlled with the `inline` attribute: `#[inline(never)]`, `#[inline]` for a standard (though very strong) inline hint, and `#[inline(always)]`. Note that `#[inline]` is required for any cross-crate inlining.
Intermediate Representation | LLVM IR Code. It can be seen in text form by passing`-S --emit-llvm` to rustc.
IR | See _Intermediate Representation_.
IRFY | Is Rust Fast Yet. Graphs tracking [how long the buildbots take to build + test](http://huonw.github.io/isrustfastyet/buildbot/). Also see its companion, [Is Rust Slim Yet](http://huonw.github.io/isrustfastyet/mem/).
lifetime | A piece of metadata that borrowed pointers have. They reference the scope that the pointer is valid for.
<a name="macro"/>macro | A type of syntax extension, defined with `macro_rules! name_of_macro`. Are a way of declaratively generating Rust from arbitrary tokens. For example, `assert!`, `debug!`, and `fail!` are macros. The standard macros can be found [here](https://github.com/mozilla/rust/blob/master/src/libsyntax/ext/expand.rs#L806).
managed pointer | `@T`, a pointer to an immutable, garbage-collected box. Also `@mut`, a pointer to a mutable, garbage-collected box.
monomorphise | The act of generating specialized versions of generic constructs at compile time to improve run time performance. See [_Whole-Program Compilation in MLton_](http://mlton.org/References.attachments/060916-mlton.pdf) and Niko Matsakis's answer on [Stackoverflow](http://stackoverflow.com/a/14198060).
newtype struct | A [*tuple structure*](#tuple_structure) with a single unnamed field. For example `struct NodeIndex(int)`. Useful to create wrapper types.
owned pointer | `~T`, a pointer to an owned box.
<a name="phantom_type" />phantom type | An `enum` with no variants. This cannot be constructed in safe code. See the [cheetsheet](https://github.com/mozilla/rust/wiki/Doc-FAQ-Cheatsheet#how-do-i-express-phantom-types) for an example.
raw pointer | `*T`, a pointer to anything. Requires unsafe code to dereference, no static verification is done on them.
rustdoc | The Rust documentation generator.
rustc | The Rust source code compiler.
rusti | The Rust interactive environment.
rustpkg | The official package manager for Rust programs and libraries.
sigil | A character placed in front of a type, identifier or literal. In the context of Rust, this usually refers to the pointer symbols: `&`, `~`, `@`, and `*`.
<a name="record_structure" />record structure | A struct declared with named fields, for example `struct Point { x: f32, y: f32 }`
stack allocation | All local variables are a stack allocation.
syntax extension | Code generation at compiletime, broken into three groups: declarative macros, which are described above as `macros`; procedural macros, which are written as Rust code that processes a token tree and produce an AST (currently requires editing the compiler), and attributes.
task | Rust's fundamental unit of computation. Similar to a thread but far more lightweight.
trait | Rust's approach to ad-hoc polymorphism, and used for generics and dynamic dispatch. Also known as [type classes](http://en.wikipedia.org/wiki/Type_class).
<a name="tuple_structure" />tuple structure | A struct declared [without named fields](#record_structure), for example `struct Point(f32, f32)`
TWiR | This Week in Rust. cmr's [weekly summary](http://cmr.github.io/blog/categories/this-week-in-rust/) of Rust's development.
<a name="type_hint" />type hint | A syntax like `foo::<int>()` to give an explicit type for a generic function, method or struct constructor. Usually redundant due to type inference.
<a name="unit_type" />unit type | The unit type, denoted `()`, permits only one value, also denoted `()`. Functions without return value have return type `()`. Sometimes called *nil*.
unit structure | A struct that only has one value, for example `struct Foo;` where `Foo` becomes the name of both the type and its only value. Works just the same as the *unit type* `()`, except it is distinct.
<a name="variant" />variant | One of the set of possible values that can be represented by an `enum`.
<a name="zero_variant_enum" />zero-variant enum | See [*phantom type*](#phantom_type).

## Syntaxicon

What is the syntax called?

syntax | name
-------|-----------
`()` | The [*unit type*](#unit_type).
`'static`, `'self`, `'a` | A *lifetime*. May also be used to name a `loop` or a `for` clause.
`~T` | If `T` is a type, an *owned pointer* to `T`.
`&T`, `&'a T` | If `T` is a type, a [*borrowed pointer*](#borrowed_pointer) to `T`, possibly with a *lifetime*.
`~fn()` | An owned [*closure*](#closure).
`&fn()` | A borrowed [*closure*](#closure). 
`&[T]`, `&'a [T]` | A vector *slice* with element type `T` and possibly with a *lifetime*.
`~[T]` | A vector.
`[T, ..n]` | A *fixed length vector* of length `n`.
`~Trait:Send` | A *trait object* where `:Send` are the *trait bounds*.
`foo!()` | Either a [*macro*](#macro) or *syntax extension*.
`#[xyz]` | An [*attribute*](#attribute).
`::<int>`, `: int` (in `let x: int ...`) | A [*type hint*](#type_hint). 
