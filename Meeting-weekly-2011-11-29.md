## Attending

Brian, Dave, Marijn, Niko, Patrick

## Type classes

* Marijn: I don't want to require names for `impl`s
* Patrick: names are extremely important for disambiguation
* Marijn: in my proposal, `impl`s are named in a separate namespace, and they default to the name of their corresponding `iface`
* Patrick: what about more duck-typing examples like:

``` rust
    impl const_vec : vec<const T> {
        fn len() -> uint { ... }
        fn push(T x) { ... }
        fn pop() { ... }
        fn get(uint i) -> T { ... }
        fn set(uint i, T x) { ... }
        ...
    }
    
    fn mylen<T : len>(x : T) -> uint {
        ret x.len();
    }
    
    fn main() {
        let v = [ 1, 2, 3 ];
        show mylen(v);
    }
```

* Marijn: I don't like automatic creation of vtables; feels magic, and not hard to declare explicitly. but as long as I *can* declare I'm not too unhappy
* Patrick: having a static assertion you conform to an iface is fine
* Niko: what about taking something that implements A and adding a couple methods?
* Dave: Haskell has this, in the form of extension
* Marijn: not in my proposal, but we'll probably want it
* Dave: could be layered
* Marijn: so are we getting close to what Go does here?
* Patrick: yes, but they don't group their `impl`s, they just infer them from scope
* Marijn: that seems nice and simple
* Dave: what I don't like about that is that it couples the caller's scope to the callee's requirements
* Patrick: yes, that crystallizes what I don't like about that approach
* Marijn: yeah, that will probably lead to all sorts of conflicts

## Fixed-size arrays

* Niko: I'm not yet convinced; sort of 50% sold. hesitation: worried it's inconvenient, and modern languages should have flexible, convenient collections
* Niko: OTOH, cheap and easy to control
* Niko: only alternative I see is flexible lists/vectors that don't allow references inside them
* Brian: can't regions help?
* Niko: it's hard
* Patrick: my view: it'd be nice if stdlib collections are really convenient
* Dave: so maybe start as libs, find the pain points, and do syntax on top of that
* Patrick: places that want syntactic convenience: construction, get, and put
* Niko: operator overloading is doable
* Marijn: can desugar to standard operations
* Dave: overloading constructors is possible too, e.g. with a common construction operator that uses e.g. a record literal
* Niko: or could be a variant of the `new` syntax
* Dave: but that requires overloading on the result type
* Marijn: we already do this and seems to be ok
* Niko: overloading is nice if you want user-defined collections to be first-class citizens
* Marijn: and you'll want the low-level construct (fixed-size arrays) for a systems language, so you can drop as low as you need
* Patrick: also for C interop
* Niko: still an impedance mismatch b/c of the length field
* Niko: could use the user-contributed c_vec module approach for interop, but still requires marshalling from C, so not really a win
* Dave: so pulse of the group?
* Patrick: I think we're all in favor of fixed-size arrays
* Niko: should be inline, see how painful it gets
* Marijn: but then you have to box them or put in other abstractions
* Niko: or `&[...]`
* Marijn: what about producing arrays
* Niko: well, `new` -- that's the other proposal
* Patrick: needs an allocator since don't have static size
* Marijn: that gets complicated awfully fast
* Dave: well, you can return a box, too
* Marijn: wait, you can't put fixed-size arrays in records!
* Niko: we could support the "C struct hack" where the last field of an array can be dynamically sized
* Marijn: so can we leave all this till after the first release?
* Patrick: yes, leave till after
* Dave: Niko, were you including static-length arrays too? this is pretty weak when you can't do any static operations on these integers
* Niko: I wasn't really including. D has a whole compile-time interpeter for manipulating lengths
* Niko: I'm inclined not to support it
* Dave: yes, let's leave it out unless we find we really need it
