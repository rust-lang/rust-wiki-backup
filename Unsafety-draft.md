This is always under debate. Here are some things that are currently considered unsafe:

* Anything that can cause segfaults, invalid reads (memory unsafety)
* Anything that can create invalid utf8 strings
* Anything that can invoke undefined behavior (ptr::offset)
* Data races
* Deadlocks (in extra::arc)

Things that are not typically considered unsafe:

* Deadlocks
* Reading data from private fields (`std::repr`, `format!("{:?}", x)`)