h2. First-class, mutable, interior slots

The runtime representation of a `slot.<T>` is bit-for-bit identical to the runtime representation of a `T`. This means that a parametric container can be given mutable and immutable variants by instantiating with an immutable element type `T` and a mutable element type `slot.<T>`.

Providing first-class support for mutable, interior slots makes it possible to create mutable vectors without having to have two different kinds of vectors. This allows us to write a simple, polymorphic map:

```
fn map.<T,U>(f : &fn(T) -> U, v : T[]) : U[] {
    ...
}
```

There is one single implementation, and the immutable version will produce an immutable vector, and the mutable version will produce a mutable version.

h3. Collapsing nested slots

h3. Type checking

h3. Code generation
