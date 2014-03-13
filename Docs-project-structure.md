Project structure in Rust is pretty free-form. The current convention is to have all source code in a `src/` directory. `src/lib.rs` is the crate root for libraries and `src/main.rs` is the crate root for executables.

Examples go in `src/examples` and out-of-crate tests (*not* using the [built-in unit testing](http://static.rust-lang.org/doc/master/guide-testing.html)) go in `src/tests`.