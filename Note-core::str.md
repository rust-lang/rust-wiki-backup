The following describes various changes in progress within core::str as of February 7, 2012.  (Beyond lots of optimization...)

###Transforming strings
**replace**: Add more test cases to verify UTF-8 safety.

###Comparing strings
**hash**: Implement murmur or cityhash to randomize this (and also other string hashes used elsewhere), see issue [1616](https://github.com/mozilla/rust/issues/1616)

###Searching
**find**: Add more test cases to verify UTF-8 support.  Should this return a byte or a char position?  Furthermore, shouldn't **find** and **contains** use an option type, rather than -1 vs. a valid value?

###String properties
Rename:

* byte_len -> len_bytes
* char_len -> len_chars / len

###Misc
Rename:

* byte_len_range -> substr_len_bytes
* char_len_range -> substr_len_chars

**sbuf**: Replace with `ctypes::c_char` per [issue 1715](https://github.com/mozilla/rust/issues/1715)

##Long term thoughts
RFCs and brainstorming:

* Input and output in other encodings should be supported: https://github.com/mozilla/rust/issues/1771
* String literals should be constant: https://github.com/mozilla/rust/issues/879
* String literals should have their types inferred, perhaps the way Haskell's -XOverloadedStrings allows ByteString, Text, and String literals to all be written like `let x = "hello"`.
* a library with ICU bindings would be nice. See ```std::unicode.rs``