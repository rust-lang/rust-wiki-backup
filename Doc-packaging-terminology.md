Some terminology:

**package**: What `rustpkg` installs (or, *will* install, once 0.6 is out). Analogous to a CPAN distribution. A package is a distribution of crates.

**crate**: A compilation unit. Compiling one .rs file may pull in other .rs files, but the end result is one crate. `rustc` compiles one crate at a time, producing either a library or an executable. A crate may span one or more .rs files. More about [crates in the manual](http://static.rust-lang.org/doc/master/rust.html#crates-and-source-files).

**module**: The Rust namespace is arranged into a hierarchy of modules. Each source (.rs) file may contain one or more modules.

**library**: Another name for a crate.

See also the [Modules and Crates section of the
tutorial](http://static.rust-lang.org/doc/master/tutorial.html#crates-and-the-module-system).


# Legacy

The `cargo` command and
[cargo-central](https://github.com/mozilla/cargo-central) are legacy
(or will be once 0.6 is released).
