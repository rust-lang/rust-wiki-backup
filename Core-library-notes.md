# Core Library

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

* core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
* everything in core will be automatically imported into the crate you're building, again, unless you pass --no-core

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

## TODO

* "char":
  * rename
      * "to_lowercase" => "to_lower" (analgous to "str")
      * "to_uppercase" => "to_upper" (analgous to "str")
* "str":
  * rename:
      * "byte_len" to "len_bytes" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
      * "char_len" to "len_chars" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
  * add:
      * "iter_lines"
      * "iter_words"
  * when to return a list of strings and when an iterator? for example "split" would be a condidate for returning an iterator when applied to a huge string