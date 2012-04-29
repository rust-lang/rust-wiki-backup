The idea is to allow a method to be extracted - as a function - from an impl that is scope.

For example, given this currently valid code:

    iface hash {
        fn hash () -> uint;
    }

    iface equals {
        fn equals (other: self) -> bool;
    }

    impl of hash for uint {
        fn hash () -> uint {
            uint::hash(self)
        }
    }

    impl of equals for uint {
        fn equals (other: uint) -> bool {
            self == other
        }
    }

A hashmap constructor function could be defined that takes the hashfn and eqfn from the scope of the function call, so they dont have to be given explicitly:

    fn hashmap <K:hash equals copy, V:copy> () -> std::map::hashmap<K, V> {
        let hashfn = extract hash<K>.hash;     // hashfn has type `fn(K) -> uint`
        let eqfn   = extract equals<K>.equals; // eqfn has type `fn(K, K) -> bool`
        ret std::map::hashmap(hashfn, eqfn);
    }

In the above code (which is not currently valid), there is a new keyword called `extract`. The expression `extract path-to-iface<type>.method` extracts a method from whichever impl is in scope for iface<type> (if any). The method is extracted as a function with self added as a new argument before the method's existing arguments.

The `hashmap` function could then be called like:

    let map = hashmap();
    map.insert(4u, "abc");
