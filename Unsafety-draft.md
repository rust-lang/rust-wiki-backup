### Things that are currently considered unsafe

* Anything that can invoke undefined behavior (ptr::offset)
* Data races
* Deadlocks (in extra::arc)
* Dereferencing a raw pointer (may be null or dangling)
* Reads of [undef](http://llvm.org/docs/LangRef.html#undefined-values) (uninitialized) memory
* Mutating an immutable value/reference, if it is not marked as non-`Freeze`

Invalid values in primitive types, even in private fields/locals:

* Dangling/null pointers in non-raw pointers
* A value other than `false` (0) or `true` (1) in a `bool`
* A discriminant in an `enum` not included in the type definition
* A value in a `char` which is a surrogate or above `char::MAX`
* non-UTF-8 sequences bytes in a `str`

### Things that are not typically considered unsafe

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)