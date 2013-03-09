This is a summary of some things that are disallowed in unsafe blocks, but is in no way exhaustive.

* Any resource management (memory, files, sockets, database connections, etc.) needs to be wrapped in an object with a destructor, or it will leak if the stack unwinds. Exposing a `close` method and never forgetting to call it isn't enough.
* You should be very aware of stack unwinding. If a failure happens, destructors will be called, so objects can't be left in a state where that could be harmful unless it can be guaranteed to never happen.
* An enum (including C-style ones) must only be set to one of the explicitly included discriminant values.
* The rules surrounding mutability should **not** be broken. If you mutate an object in a method, you can't hide that without using `@mut` pointers as wrappers. You can also use `*mut`, but the pointer is still assumed to fall under the mutability rules (just not what it points to).
* You should not work around the restrictions placed on `@`, `~` and `&`, instead use `*` pointers if you can't fit something into the semantics of the safe pointer types.