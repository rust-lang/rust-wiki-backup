# Core Library

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

   - core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
   - everything in core will be automatically imported into the crate
     you're building, again, unless you pass --no-core