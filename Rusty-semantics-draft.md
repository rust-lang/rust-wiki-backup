## Freezing

Borrowing an immutable pointer to an object freezes it and prevents mutation. `Owned` objects have freezing enforced statically at compile-time. Mutable managed boxes handle freezing dynamically when a borrow is done to anything contained in them, and will throw a failure if an attempt to modify them is made while they are frozen.