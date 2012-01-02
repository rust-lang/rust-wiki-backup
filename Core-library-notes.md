# Core Library

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

* core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
* everything in core will be automatically imported into the crate you're building, again, unless you pass --no-core

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

## TODO

* "str":
  * rename "byte_len" to "len_bytes" and "char_len" to "len_chars", to be in line of "iter_chars", "loop_chars", "push_byte", "push_char"
  * rename "char_len" => "len" and "iter_chars" => "iter" because operating on chars would be most common for users?
  * what functions are missing?
  * when to return a list of strings and when an iterator?