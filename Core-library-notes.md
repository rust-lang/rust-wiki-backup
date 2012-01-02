# Core Library

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

* core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
* everything in core will be automatically imported into the crate you're building, again, unless you pass --no-core

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

## TODO

* "char":
  * fix "maybe_digit" dcoumentation "Function: to_digit"
  * rename:
      * "to_lowercase" => "to_lower" (analgous to "str")
      * "to_uppercase" => "to_upper" (analgous to "str")
  * add:
      * "is_digit"
* "str":
  * add "pure" to a lot of functions
  * rename:
      * "str_from_cstr" => "from_cstr"
      * "byte_len" => "len_bytes" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
      * "byte_len_range" => "len_bytes_range"
      * "char_len" => "len_chars"
      * "char_len_range" => "len_chars_range"
      * "unsafe_from_byte" => "from_byte_unsafe"
      * "unsafe_from_bytes" => "from_bytes_unsafe"
      * "split_str" => "split_char"? (analogous to "split_str")
  * add:
      * "len" := "len_char"
      * "map" and implement "to_upper", "to_lower" on top of it
      * "iter_lines"
      * "iter_words"
  * when to return a list of strings and when an iterator? for example "split" would be a condidate for returning an iterator when applied to a huge string