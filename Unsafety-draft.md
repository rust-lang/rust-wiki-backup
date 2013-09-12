### Things that are currently considered unsafe

* Data races
* Deadlocks (in extra::arc)
* Dereferencing a null/dangling raw pointer
* Mutating an immutable value/reference, if it is not marked as non-`Freeze`
* Reads of [undef](http://llvm.org/docs/LangRef.html#undefined-values) (uninitialized) memory
* Anything considered undefined behaviour by LLVM, such as indexing more than one byte past the end of an object with `ptr::offset`

Invalid values in primitive types, even in private fields/locals:

* Dangling/null pointers in non-raw pointers
* A value other than `false` (0) or `true` (1) in a `bool`
* A discriminant in an `enum` not included in the type definition
* A value in a `char` which is a surrogate or above `char::MAX`
* non-UTF-8 byte sequences in a `str`

### Things that are not typically considered unsafe

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)