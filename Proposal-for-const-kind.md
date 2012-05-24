This describes a proposal for a const kind. It should be pretty straightforward, but it seems good to have some documentation about the const kind somewhere other than the kind checker.

# Motivation

The primary use case at the moment is to facilitate safe sharing of immutable data between tasks. We have a first step towards this in the `std::arc` module (https://github.com/mozilla/rust/blob/master/src/libstd/arc.rs). The main problem is that our type/kind system is not strong enough to enforce the what is needed to make this safe. Basically, `std::arc` still allows tasks to end up sharing mutable data, but using the `const` kind will allow Rust to enforce deep immutable data.

We believe that deep immutability will be useful in other cases as well.

# Implementation

The recent kind system changes mean this involves adding a new const bit to the kind masks. The more important part is what types fall into what kinds. I propose these rules:

1. Scalar types are const.
2. Types that contain another type (for example, records or vectors) are not const if any of their contained types are not const.
3. Vectors are const if they are not mutable vectors.
   * Is this true across all types of vectors?
3. Strings are const.
4. Records are const provided none of the fields are mutable.
5. Enums are const provided no variants contain non-const elements.
6. Tuples are const provided they are only made up of const things.
7. Classes are const, provided they follow the same rules as records.
8. Unique pointers and shared boxes are const.
   * But shared boxes are still not sendable.
8. Bare `fn`s are const.
9. Resources are not const
10. Ifaces and closures are not const.

As far as syntax goes, I suggested adding `const` as a kind bound. For example:
```
fn foo<T: const>(x: T) { ... }
```

# Notes and future directions

The const kind by itself is not enough to ensure safe sharing. Instead, we'll have to parameterize `arc` over `send const` types.

It would be nice to come up with a notion of const closures. Const closures is a more invasive change, so we'll save this for later.