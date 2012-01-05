# Interfaces

This describes the interface (`iface`) and implementation (`impl`) features as they exist at this point (Jan 5 2012). I've mostly copy-pasted https://github.com/graydon/rust/wiki/Interface-and-Implementation-Proposal and adjusted the parts that changed or haven't materialized (yet).

## Method implementation

    // An implementation named `iter_util` for vector types
    impl iter_util<T> for [T] {
        fn len() -> uint { std::vec::len(self) }
        fn iter(f: block(T)) { for elt in self { f(elt); } }
        fn map<U>(f: block(T) -> U) -> [U] {
            let rslt = [];
            for elt in self { rslt += [f(elt)]; }
            rslt
        }
    }

Note that both the `impl` and the individial methods can take type parameters.

## Static resolution

If the above `impl` was defined in `std::vec`, another module can import the interface implementation and use it for method calls:

    import std::vec::iter_util;
    [1, 2, 3].iter {|e| log_err e;}

This call is statically resolved. The set of imported interface implementations is searched for an `iter` method matching the type of the method call's receiver (`[int]`). If more than one match, the 'nearest' (in scope) import wins. If there are still multiple implementations in the nearest scope, an error is raised (we may want to revisit that and use specificity scoring).

## Declared interfaces

    iface seq<T> {
        fn len() -> uint;
        fn iter(block(T));
    }

The above declares a named interface type `seq`. You can make implementations refer to a specific interface by referring to it using the `of` keyword.

    impl list_seq<T> of std::seq::seq<T> for std::list::list {
        fn len() -> uint { std::list::len(self) }
        fn iter(f: block(T)) { std::list::iter(self, f); }
    }

You are allowed to leave off the name of the `impl` in order to inherit the name of the `iface`, as in `impl <T> of std::seq::seq<T> for ...`. Impl names don't clash -- they are only used for importing and exporting, and if you import a name that happens to refer to multiple impls, you simply get all of them  in scope.

## Dynamic dispatch through bounded type params

If `seq` names an interface that has been imported, it can be used as a bound on a type parameter, allowing methods in that interface to be used inside the generic function.

    // Declare T to be an instance of the seq interface
    fn total_len<T:seq>(seqs: [T]) -> uint {
        let cnt = 0u;
        for s in seqs { count += s.len(); }
        count
    }

You can also specify multiple bounds (separated by whitespace, as in `<T:seq to_str>`). What actually happens here is that vtables for the interface are passed as hidden parameters to the generic (much like type descriptors).

## Kinds as bounded type parameters

The `copy` and `send` kinds are now annotated just like other parameter bounds, removing a special case from our syntax. They behave very much like interfaces, except that the do not refer to actual impls, but instead allow the compiler to do things with the type parameter that it normally wouldn't. (No actual dictionary is passed for such bounds, they are simply checked statically.)
