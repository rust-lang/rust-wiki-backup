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
  * rename:
      * "str_from_cstr" => "from_cstr" renamed from
      * "to_chars" => "chars" renamed from (analogous to "bytes")
      * "unsafe_from_byte" => "from_byte_unsafe" renamed from
      * "unsafe_from_bytes" => "from_bytes_unsafe" renamed from
  * goal:
      * "bytes_iter": added
      * "chars_iter" renamed from "iter_chars" (to make the analagous to the iter versions to come)
      * "chars_loop" renamed from "loop_chars"
      * "len": added := "len_char"
      * "len_bytes" renamed from "byte_len" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
      * "len_chars" renamed from "char_len"
      * "lines": added
      * "lines_iter": added
      * "map" and implement "to_upper", "to_lower" on top of it
      * "split": added  := "split_char"
      * "split_char" renamed from "split" (analogous to "split_str")
      * "split_iter": added
      * "substr_chars_loop" renamed from "loop_chars_sub" (analogous to "substr")
      * "substr_len_bytes" renamed from "byte_len_range"
      * "substr_len_chars" renamed from "char_len_range"
      * "words": added
      * "words_iter": added
  * add "pure" to a lot of functions