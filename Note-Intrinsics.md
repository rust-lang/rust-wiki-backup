Intrinsics are implemented in Rust as special functions that directly generate LLVM IR in their bodies. This allows for very specific code to be generated which is useful when implementing very low level features.

However, adding new intrinsics can be tricky. There are 3 places that you need to alter in order to add an intrinsic: `typeck/check/mod.rs`, `trans/foreign.rs` and `trans/type_use.rs`. All three places need to be updated in order for the intrinsic to be available. In order to use the intrinsic in code, you then need to declare it with the correct signature and the abi "rust-intrinsic".

### In Typeck

In `typeck/check/mod.rs` there is a function called `check_intrinsic_type`, inside there is a match statement that matches against the name of the intrinsic. Look at the other examples to get an idea of what you need to enter here. This is where the type checker checks your function declaration in the source, so you need to make sure they match. This is because intrinsics are often declared where ever they are needed, so the types all need to match

### In Foreign

This is where rust translates the foreign declaration into LLVM code. Here you can again look at the surrounding code to get an idea of what to enter.

### In Type Use

This is where rust determines if it can combine function bodies for generic functions. This means you need an entry for your intrinsic, so rust knows it exists.