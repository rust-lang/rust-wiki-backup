Status of changes in progress, by section (per the [naturaldoc](http://doc.rust-lang.org/doc/core/files/str-rs.html)), as of Superbowl 2012.

### Creating a string
**from_cstr_len**: Add this per [issue 1666](https://github.com/mozilla/rust/issues/1666).

###Adding to and removing from a string
-

###Transforming strings
This renaming [is pending](https://github.com/mozilla/rust/pull/1754):

* split -> split_byte
* splitn -> splitn_byte
* split_func -> split
* split_char -> split_char
* split_chars_iter -> split_char_iter

After these pending changes I think split, split_char, split_byte, and split_str should all behave identically in terms of "", "x", "xABCx", "xx", and so on.

**substr**: Currently not UTF-8.  Should we (A) make substr (char) and substr_bytes, or (B) just make substr a char function?

**splitn_char**: Add this.

**replace**: Add more test cases to verify UTF-8 safety.



###Comparing strings
**hash**: Implement murmur or cityhash to randomize this (and also other string hashes used elsewhere), see issue [1616](https://github.com/mozilla/rust/issues/1616)

###Iterating through strings
**splitn_char_iter**: This needs to use splitn_char

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

###Unsafe
-
