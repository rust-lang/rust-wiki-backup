## Destructors

Rust uses destructors to handle the release of resources like memory allocations, files and sockets. An object will only be destroyed when there is no longer any way to access it, which prevents dynamic failures from an attempt to use a freed resource. When a task fails, the stack unwinds and the destructors of all objects owned by that task are called.

## Ownership

An object's owner is responsible for managing the lifetime of the object by calling the destructor, and it determines whether the object is mutable. Ownership is recursive, so mutability is inherited recursively and a destructor will destroy the contained tree of owned objects. Variables are top-level owners and destroy the contained object when they go out of scope. If an object consists entirely of a single ownership tree, it is given the `Owned` trait and can be sent between tasks.

## Borrowed pointers

A borrowed pointer is a reference to an existing object, and expresses no ownership over it. An immutable borrowed pointer (`&`) freezes the object as long as it exists. A mutable borrowed pointer (`&mut`) is a unique mutable reference to a mutable object.

## Owned boxes

An owned box is a uniquely owned allocation on the heap. An owned box inherits the mutability and lifetime of the owner as it would if there was no box. The purpose of an owned box is to add a layer of indirection in order to create recursive data structures or cheaply pass around an object larger than a pointer.

## Managed boxes

Managed boxes lack an owner, so they are a break in the ownership tree and don't inherit mutability. They do own the contained object, and mutability is defined by the type of the shared box (`@` or `@mut`). An object containing a managed box is not `Owned`, and can't be sent between tasks. Managed boxes have their lifetimes managed by a task-local garbage collector and will be destroyed at some point after there are no more references to them or the end of the task.

## Freezing

Borrowing an immutable pointer to an object freezes it and prevents mutation. `Owned` objects have freezing enforced statically at compile-time. Mutable managed boxes handle freezing dynamically when a borrow is done to anything contained in them, and will throw a failure if an attempt to modify them is made while they are frozen.

## Moving

Passing a parameter by-value, assigning by-value and returning by-value is a shallow copy. A shallow copy is a move of ownership for any type with an ownership tree containing an owned box or type with a custom destructor.