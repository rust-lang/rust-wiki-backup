# Core Library

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

* core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
* everything in core will be automatically imported into the crate you're building, again, unless you pass --no-core

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

## TODO

* "char":
  * fix "maybe_digit" documentation "Function: to_digit"
  * rename:
      * "to_lowercase" => "to_lower" (analogous to "str")
      * "to_uppercase" => "to_upper" (analogous to "str")
  * add:
      * "is_digit"
* "str":
  * add "pure" to a lot of functions
  * rename:
      * "byte_len" => "len_bytes" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
      * "byte_len_range" => "substr_len_bytes"
      * "char_len" => "len_chars"
      * "char_len_range" => "substr_len_chars"
      * "iter_chars" => "chars_iter" (to make the analagous to the iter versions to come)
      * "loop_chars" => "chars_loop"
      * "loop_chars_sub" => "substr_loop" (analogous to "substr")
      * "split" => "split_char"? (analogous to "split_str")
      * "str_from_cstr" => "from_cstr"
      * "to_chars" => "chars" (analogous to "bytes")
      * "unsafe_from_byte" => "from_byte_unsafe"
      * "unsafe_from_bytes" => "from_bytes_unsafe"
  * add:
      * "len" := "len_char"
      * "split" := "split_char"
      * "bytes_iter"
      * "lines"
      * "lines_iter"
      * "split_iter"
      * "words"
      * "words_iter"
      * "map" and implement "to_upper", "to_lower" on top of it