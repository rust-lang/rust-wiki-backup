This is currently a work in progress - feel free to alter or add definitions as necessary.

term | definition
-----|-----------
bikeshed | A [highly important discussion](http://www.catb.org/jargon/html/B/bikeshedding.html) about some non-fundamental part of the language (such as syntax or identifier names).
borrowed pointer |
bors | A [Python script](https://github.com/graydon/bors) that checks for reviewed pull requests and runs the test on it, merging it if they pass.
box |
closure | Refers both to the type (`&fn`, `~fn`) and the literal notation: `|args| expression` (where expression can be a block, ie `|x| { println(x.to_str()); 5}`). It is said to "close over" its environment; it can "capture" values from surrounding code.
crate |
FFI | See _foreign function interface_.
foreign function interface |
heap allocation | A dynamic allocation performed either by `~` or `@`, which call to `malloc` in the default runtime (which is a statically-linked `jemalloc`)
inline |
lifetime |
macro |
managed pointer |
monomorphise | The act of generating specialized versions of generic constructs at compile time to improve run time performance. See [_Whole-Program Compilation in MLton_](http://mlton.org/References.attachments/060916-mlton.pdf) and Niko Matsakis's answer on [Stackoverflow](http://stackoverflow.com/a/14198060).
owned pointer |
raw pointer | 
rustdoc | The Rust documentation generator.
rustc | The Rust source code compiler.
rusti | The Rust interactive environment.
rustpkg | The official package manager for Rust programs and libraries.
sigil | A character placed in front of a type, identifier or literal. In the context of Rust, this usually refers to the pointer symbols: `&`, `~`, `@`, and `*`.
stack allocation | 
syntax extension |
task | Rust's fundamental unit of computation. Similar to a thread but far more lightweight.
trait | Rust's approach to ad-hoc polymorphism, and used for generics and dynamic dispatch. Also known as [type classes](http://en.wikipedia.org/wiki/Type_class).