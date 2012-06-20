Rust provides several intrinsic functions. These are called like normal functions, but are translated directly into LLVM code by trans. These are meant to implement low-level, unsafe things in the core libraries. This page attempts to list all the intrinsics, what they do, how to use them, and why they exist. The intrinsics are currently implemented in src/rustc/middle/trans/native.rs.

* `size_of`
* `move_val`
* `move_val_init`
* ...