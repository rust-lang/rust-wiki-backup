The middle end includes the resolver, typechecker, kind checker, typestate checker, and various other semantic analysis passes. The source files for it live in ```src/librustc/middle```. The middle end also includes the code generator that generates LLVM code from Rust abstract syntax tree: the code generator files are in ```src/librustc/middle/trans```.

* [[Note Intrinsics]]