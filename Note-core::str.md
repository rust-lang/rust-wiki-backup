The following describes various changes in progress within core::str as of February 7, 2012.  (Beyond lots of optimization...)

### Chars vs Bytes
How much do we need to expose the byte-vector internals?
I'd prefer a mostly "string of chars".

See #1849

###Transforming strings
**replace**: Add more test cases to verify UTF-8 safety.

###Comparing strings
**hash**: Implement murmur or cityhash to randomize this (and also other string hashes used elsewhere), see issue [1616](https://github.com/mozilla/rust/issues/1616)

###Misc
**sbuf**: Replace with `ctypes::c_char` per [issue 1715](https://github.com/mozilla/rust/issues/1715)

##Long term thoughts
RFCs and brainstorming:

* Input and output in other encodings should be supported: https://github.com/mozilla/rust/issues/1771
* String literals should be constant: https://github.com/mozilla/rust/issues/879
* String literals should have their types inferred, perhaps the way Haskell's -XOverloadedStrings allows ByteString, Text, and String literals to all be written like `let x = "hello"`.
* a library with ICU bindings would be nice. See ```std::unicode.rs```