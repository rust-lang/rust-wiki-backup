### Things that are currently considered unsafe

* Anything that can cause segfaults (memory unsafety)
* Reads of undefined memory (http://llvm.org/docs/LangRef.html#undefined-values), even for plain old data
* Anything that can create invalid utf8 strings
* Anything that can invoke undefined behavior (ptr::offset)
* Data races
* Deadlocks (in extra::arc)

Storing invalid values in certain types, even in private fields:

* Storing dangling/null pointers in non-raw pointers
* Storing a value other than `false` (0) or `true` (1) in a `bool`
* Storing a discriminant in an `enum` not included in the type definition
* Storing a value in a `char` which is a surrogate or above `char::MAX`

### Things that are not typically considered unsafe

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)

## Questionable

* shifts by more than the number of bits in the value (LLVM calls the result *undefined*, which may cause soundness issues if it's actually treated as `undef`)