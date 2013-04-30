Rust provides several intrinsic functions. These are called like normal functions, but are translated directly into LLVM code by trans. These are meant to implement low-level, unsafe things in the core libraries. This page attempts to list all the intrinsics, what they do, how to use them, and why they exist. The intrinsics are currently implemented in src/rustc/middle/trans/native.rs.

* `addr_of`
* `atomic_add`
* `atomic_add_acq`
* `atomic_add_rel`
* `atomic_sub`
* `atomic_sub_acq`
* `atomic_sub_rel`
* `atomic_xchg`
* `atomic_xchg_acq`
* `atomic_xchg_rel`
* `forget`
* `frame_address`
* `get_tydesc`
* `init`
* `min_align_of`
* `move_val`
* `move_val_init`
* `needs_drop`
* `pref_align_of`
* `reinterpret_cast`
* `size_of`
* `visit_ty`
