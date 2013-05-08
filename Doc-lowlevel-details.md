Dispatch
========

Type-parameterized functions are specialized for each set of type parameters used with it in a crate. The method calls are resolved statically at compile-time. Example:

```rust
fn foo<T: ToStr>(x: &T) -> ~str { x.to_str() }
foo(&5u); // `foo<uint>` is instantiated for this crate
```
Using traits as objects is dynamic dispatch at runtime. The method calls are resolved by dereferencing a pointer to a vtable at runtime and then calling a function pointer. Example:

```rust
fn foo(x: &ToStr) -> ~str { x.to_str() }
foo(&5u as &ToStr); // a vtable is generated for `uint` with the `to_str` method
```
At runtime, `&ToStr` is represented as a a pointer to a vtable with all the methods in the trait (just `to_str` in this case) and a pointer to the value. The pointer to the value is the pointer sigil in front of the trait object.