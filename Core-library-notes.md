# Core Library

The Core Library (libcore) complements the Standard Library (libstd) in the following fashion:

* core is automatically 'use'd in any crate, unless you ask for --no-core when running the compiler.
* everything in core will be automatically imported into the crate you're building, again, unless you pass --no-core

See https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html

## TODO

* "char":
  * add:
      * "is_ascii"
      * "is_digit"

* "str":
      * "all_chars" **rename** from "loop_chars"
      * "any_char" **add** (analogous to "all_chars")
      * "bytes"
      * "bytes_iter": **add**
      * "char_at"
      * "char_range_at"
      * "chars" **rename** from "to_chars" (analogous to "bytes")
      * "chars_iter" **rename** from "iter_chars" (to make the analagous to the iter versions to come)
      * "concat"
      * "connect"
      * "contains"
      * "ends_with"
      * "escape"
      * "escape_char"
      * "eq"
      * "find"
      * "from_byte" **rename** from "unsafe_from_byte"
      * "from_bytes" **rename** from "unsafe_from_bytes"
      * "from_char"
      * "from_chars"
      * "from_cstr"
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
      * "slice" **rename** from "safe_slice"
      * "slice_char" **rename** from "char_slice"
      * "slice_unsafe" **rename** from "slice"
      * "split": **add**, takes a block of type 'char -> bool'
      * "split_char" **add**/**rename** from "split" (analogous to "split_str")
      * "split_char_iter": **add**
      * "splitn_char": **rename** from "splitn"
      * "splitn_char_iter": **add**
      * "split_str"
      * "split_str_iter": **add**
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
      * "unshift_byte": **add**
      * "unshift_char"
      * "utf8_char_width"
      * "words": **add**
      * "words_iter": **add**
* "str":
  * "is_ascii": implement on top of "char::is_ascii", "chars_loop"
  * implement "to_upper", "to_lower" on top of "map"
  * add "pure" to a lot of functions