### Things that are currently considered unsafe

* Anything that can cause segfaults, invalid reads (memory unsafety)
* Anything that can create invalid utf8 strings
* Anything that can invoke undefined behavior (ptr::offset)
* Data races
* Deadlocks (in extra::arc)

Storing invalid values in certain types, even in private fields:

* Storing dangling/null pointers in non-raw pointers
* Storing a value other than `false` (0) or `true` (1) in a `bool`
* Storing a value in an `enum` not included as a variant
* Storing a value in a `char` which is a surrogate or above `char::MAX`

### Things that are not typically considered unsafe

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)