The following describes various changes in progress within core::str as of February 5, 2012.  (Beyond lots of optimization...)

### Creating a string
**from_cstr_len**: Add this per [issue 1666](https://github.com/mozilla/rust/issues/1666).

###Transforming strings
This renaming [is pending](https://github.com/mozilla/rust/pull/1754):

* split -> split_byte
* splitn -> splitn_byte
* split_func -> split
* split_char -> split_char
* split_chars_iter -> split_char_iter
* substr: use char positions
* splitn_char: add


After these pending changes I think split, split_char, split_byte, and split_str should all behave identically in terms of "", "x", "xABCx", "xx", and so on.

**replace**: Add more test cases to verify UTF-8 safety.

###Comparing strings
**hash**: Implement murmur or cityhash to randomize this (and also other string hashes used elsewhere), see issue [1616](https://github.com/mozilla/rust/issues/1616)

###Searching
**index** and **rindex**: These are finding a byte, not a char.  Should we create `index_byte` and `rindex_byte` in addition to making these look for a char?  Should all return a byte position, or char and byte positions, respectively?

**find**: Add more test cases to verify UTF-8 support.  Should this return a byte or a char position?  Furthermore, shouldn't **find** and **contains** use an option type, rather than -1 vs. a valid value?

###String properties
Rename:

* byte_len -> len_bytes
* char_len -> len_chars

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
* a library with ICU bindings would be nice