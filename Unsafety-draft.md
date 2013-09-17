### Things that are currently considered unsafe

This is a list of behaviour `unsafe` blocks *must prevent*. These issues cannot be caused by safe code, if all interfaces using `unsafe` code internally are correct.

* Data races
* Dereferencing a null/dangling raw pointer
* Mutating an immutable value/reference, if it is not marked as non-`Freeze`
* Reads of [undef](http://llvm.org/docs/LangRef.html#undefined-values) (uninitialized) memory
* Indexing outside of the bounds of an object with `ptr::offset`, with the exception of one byte past the end which is permitted.
* Invoking undefined behavior via compiler intrinsics (such as the offset example above)

Invalid values in primitive types, even in private fields/locals:

* Dangling/null pointers in non-raw pointers, or slices
* A value other than `false` (0) or `true` (1) in a `bool`
* A discriminant in an `enum` not included in the type definition
* A value in a `char` which is a surrogate or above `char::MAX`
* non-UTF-8 byte sequences in a `str`

### Things that are not typically considered unsafe

This is a list of behaviour not considered *unsafe* in Rust terms, but that may be undesired.

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)
* Leaks due to reference count cycles, even in the global heap
* Exiting without calling destructors
* Sending signals
* Accessing/modifying the file system
* Unsigned integer overflow (well-defined as wrapping)
* Signed integer overflow (well-defined as two's complement representation wrapping)