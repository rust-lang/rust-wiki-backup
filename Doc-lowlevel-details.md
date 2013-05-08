Dispatch
========

Generics are statically-compiled, specialized for each type. Using traits as objects is runtime dispatch. Example:

```rust
fn foo(a: ToStr);
fn bar<T: ToStr>(a: T);
```

At runtime, `foo` is represented as a "fat pointer", a pointer to a vtable for ToStr and a pointer to the data. `bar` is specialized at compile-time for every type it is used with.