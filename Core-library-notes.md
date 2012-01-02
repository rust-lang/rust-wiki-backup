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
      * "bytes"
      * "bytes_iter": **add**
      * "char_at"
      * "char_range_at"
      * "chars" **rename** from "to_chars" (analogous to "bytes")
      * "chars_iter" **rename** from "iter_chars" (to make the analagous to the iter versions to come)
      * "chars_loop" **rename** from "loop_chars"
      * "concat"
      * "connect"
      * "contains"
      * "ends_with"
      * "escape"
      * "escape_char"
      * "eq"
      * "find"
      * "from_byte_unsafe" **rename** from "unsafe_from_byte"
      * "from_bytes_unsafe" **rename** from "unsafe_from_bytes"
      * "from_char"
      * "from_chars"
      * "from_cstr" **rename** from "str_from_cstr"
      * "hash"
      * "index"
      * "is_ascii"
      * "is_empty"
      * "is_not_empty"
      * "is_utf8"
      * "is_whitespace"
      * "len":: **add** := "len_char"
      * "len_bytes" **rename** from "byte_len" (analogous to "iter_chars", "loop_chars", "push_byte", "push_char")
      * "len_chars" **rename** from "char_len"
      * "lines": **add**
      * "lines_iter": **add**
      * "lteq"
      * "map": **add**
      * "pop_byte"
      * "pop_char"
      * "push_byte"
      * "push_bytes"
      * "push_char"
      * "replace"
      * "rindex"
      * "shift_byte"
      * "shift_char"
      * "slice"
      * "slice_char" **rename** from "char_slice"
      * "slice_safe" **rename** from "safe_slice"
      * "split" := "split_char"
      * "splitn"
      * "split_char" **add**/**rename** from "split" (analogous to "split_str")
      * "split_iter": **add**
      * "split_str"
      * "starts_with"
      * "substr"
      * "substr_chars_loop" **rename** from "loop_chars_sub" (analogous to "substr")
      * "substr_len_bytes" **rename** from "byte_len_range"
      * "substr_len_chars" **rename** from "char_len_range"
      * "to_lower"
      * "to_upper"
      * "trim"
      * "trim_left"
      * "trim_right"
      * "unshift_char"
      * "utf8_char_width"
      * "words": **add**
      * "words_iter": **add**
  * implement "to_upper", "to_lower" on top of "map"
  * add "pure" to a lot of functions