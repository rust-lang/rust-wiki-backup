This is currently a work in progress - feel free to alter or add definitions as necessary.

term | definition
-----|-----------
attribute | A way of adding metadata to an item. Written as `#[attribute_name]` (other examples being `#[doc="Foo"]`, `#[doc(hidden="true")]`).
bikeshed | A [highly important discussion](http://www.catb.org/jargon/html/B/bikeshedding.html) about some non-fundamental part of the language (such as syntax or identifier names).
borrowed pointer | Also known as a "reference". It references an object without taking ownership of it. Has an associated lifetime, to assert that it is always valid.
bors | A [Python script](https://github.com/graydon/bors) that checks for reviewed pull requests and runs the test on it, merging it if they pass.
box | An allocated chunk of memory.
closure | Refers both to the type (`&fn`, `~fn`) and the literal notation: `∣args∣ expression` (where expression can be a block, ie `∣x∣ { println(x.to_str()); 5}`). It is said to "close over" its environment; it can "capture" values from surrounding code. (*Note:* This is using a non-pipe unicode character because github doesn't like using that character in tables.)
crate | Rust's compilation unit, a single library or executable. Is the root of a namespace.
FFI | See _foreign function interface_.
foreign function interface | Calling code written in another language. Rust has a native C FFI, using `extern "C" fn`.
heap allocation | A dynamic allocation performed either by `~` or `@`, which call to `malloc` in the default runtime (which is a statically-linked `jemalloc`).
inlining | Inlining is removal of a function call by including the function body directly into the callsite, enabling further optimizations. Controlled with the `inline` attribute: `#[inline(never)]`, `#[inline]` for a standard (though very strong) inline hint, and `#[inline(always)]`. Note that `#[inline]` is required for cross-crate inline.
lifetime | A piece of metadata that borrowed pointers have. They reference the scope that the pointer is valid for.
macro | A type of syntax extension, defined with `macro_rules! name_of_macro`. Are a way of declaratively generating Rust from arbitrary tokens. For example, `assert!`, `debug!`, and `fail!` are macros. The standard macros can be found [here](https://github.com/mozilla/rust/blob/master/src/libsyntax/ext/expand.rs#L806).
managed pointer | `@T`, a pointer to a mutable, garbage-collected box.
monomorphise | The act of generating specialized versions of generic constructs at compile time to improve run time performance. See [_Whole-Program Compilation in MLton_](http://mlton.org/References.attachments/060916-mlton.pdf) and Niko Matsakis's answer on [Stackoverflow](http://stackoverflow.com/a/14198060).
owned pointer | `~T`, a pointer to an owned box.
raw pointer | `*T`, a pointer to anything. Requires unsafe code to dereference, no static verification is done on them.
rustdoc | The Rust documentation generator.
rustc | The Rust source code compiler.
rusti | The Rust interactive environment.
rustpkg | The official package manager for Rust programs and libraries.
sigil | A character placed in front of a type, identifier or literal. In the context of Rust, this usually refers to the pointer symbols: `&`, `~`, `@`, and `*`.
stack allocation | All local variables are a stack allocation.
syntax extension | Code generation at runtime, broken into three groups: declarative macros, which are described above as `macros`; procedural macros, which are written as Rust code that processes a token tree and produce an AST (currently requires editing the compiler), and attributes.
task | Rust's fundamental unit of computation. Similar to a thread but far more lightweight.
trait | Rust's approach to ad-hoc polymorphism, and used for generics and dynamic dispatch. Also known as [type classes](http://en.wikipedia.org/wiki/Type_class).