Note: this is out of date and will be deleted: we now have *traits*!

***

This describes the interface (`iface`) and implementation (`impl`) features as they exist at this point (Jan 5 2012). I've mostly copy-pasted [[Proposal for interfaces]] and adjusted the parts that changed or haven't materialized (yet).

## Method implementation

The form below implements an interface-less implementation. This can be used for static overloading, but not for any of the dynamic interface features. The (export/import) name of the impl is `iter_util`, and it applies to all vector types.

    impl <T> of iter_util<T> for [T] {
        fn len() -> uint { vec::len(self) }
        fn iter(f: block(T)) { for elt in self { f(elt); } }
        fn map<U>(f: block(T) -> U) -> [U] {
            let rslt = [];
            for elt in self { rslt += [f(elt)]; }
            rslt
        }
    }

Note that both the `impl` and the individual methods can take type parameters.

## Static resolution

If the above `impl` was defined in `core::vec`, another module can import the interface implementation and use it for method calls:

    import vec::iter_util;
    [1, 2, 3].iter {|e| log(error, e);}

This call is statically resolved. The set of imported interface implementations is searched for an `iter` method matching the type of the method call's receiver (`[int]`). If more than one match, the 'nearest' (in scope) import wins. If there are still multiple implementations in the nearest scope, an error is raised (we may want to revisit that and use specificity scoring).

## Declared interfaces

    iface seq<T> {
        fn len() -> uint;
        fn iter(block(T));
    }

The above declares a named interface type `seq`. You can make implementations refer to a specific interface by referring to it using the `of` keyword.

    impl list_seq<T> of seq::seq<T> for std::list::list {
        fn len() -> uint { std::list::len(self) }
        fn iter(f: block(T)) { std::list::iter(self, f); }
    }

You are allowed to leave off the name of the `impl` in order to inherit the name of the `iface`, as in `impl <T> of seq::seq<T> for ...`. Impl names don't clash -- they are only used for importing and exporting, and if you import a name that happens to refer to multiple impls, you simply get all of them  in scope.

## Dynamic dispatch through bounded type params

If `seq` names an interface that has been imported, it can be used as a bound on a type parameter, allowing methods in that interface to be used inside the generic function.

    // Declare T to be an instance of the seq interface
    fn total_len<T:seq>(seqs: [T]) -> uint {
        let cnt = 0u;
        for s in seqs { count += s.len(); }
        count
    }

You can also specify multiple bounds (separated by whitespace, as in `<T:seq to_str>`). What actually happens here is that vtables for the interface are passed as hidden parameters to the generic (much like type descriptors).

## Per-value vtables

The above does not help when you want to have a collection of generic values, each with its own vtable. For this, you can explicitly casting a value to an interface type.

    circle(10f, "red") as drawable

This copies the value and the data (vtable, type parameters) need to call methods on it into a into a reference counted box. You could then stuff a number of those into a collection, and call their `.draw()` methods through the boxed vtable.

    fn draw_all(ds: [drawable]) { for d in ds { d.draw(); } }

## Kinds as bounded type parameters

The `copy` and `send` kinds are now annotated just like other parameter bounds, removing a special case from our syntax. They behave very much like interfaces, except that the do not refer to actual impls, but instead allow the compiler to do things with the type parameter that it normally wouldn't. (No actual dictionary is passed for such bounds, they are simply checked statically.)
