# Interface/Implementation Proposal

Discussion [here][1], issue number #1227

[1]: https://mail.mozilla.org/pipermail/rust-dev/2011-November/001010.html

## Method implementation

    // This implements an interface `iter_util` for vector types
    impl iter_util<T> : [T] {
        fn len() -> uint { std::vec::len(self) }
        fn iter(f: block(T)) { for elt in self { f(elt); } }
        fn iter_n(f: block(T, uint)) {
            let i = 0u;
            for elt in self { f(elt, i); i += 1u; }
        }
    }

## Static resolution

If that was defined in `std::vec`, another module can import the interface implementation and use it for method calls:

    import std::vec::iter_util;
    [1, 2, 3].iter {|e| log_err e;}

This call is statically resolved. The set of imported interface implementations is searched for an `iter` method matching the type of the method call's receiver (`[int]`). If more than one match, the 'nearest' import wins. If there are still multiple implementatoins in the nearest scope, we'll want to somehow score them for specificity, and give precedence to more specific implementations. This will be implemented later.

## Declared interfaces

    iface seq<T> {
        fn len() -> uint;
        fn iter(block(T));
    }

The above declares a named interface type `seq`. You can make implementations refer to a specific interface by using its name.

    impl std::seq::seq<T> : std::list::list {
        fn len() -> uint { std::list::len(self) }
        fn iter(f: block(T)) { std::list::iter(self, f); }
    }

This will recycle the name of the interface for the name of the implementation, so if you define this in `std::list`, `std::list::seq` will refer to this implementation. The compiler will verify that an implementation of an existing interface actually conforms to the interface declaration.

*We may want to allow optionally specifying a different name. I'm currently thinking that, if we allow implementations to share names as long as they are specializing on different types, it will rarely be a problem to default to the interface name.*

## Dynamic dispatch through bounded type params

If `seq` names an interface that has been imported, it can be used as a bound on a type parameter, allowing methods in that interface to be used inside the generic function.

    // Declare T to be an instance of the seq interface
    fn total_len<T:seq>(seqs: [T]) -> uint {
        let cnt = 0u;
        for s in seqs { count += s.len(); }
        count
    }

You can also specify multiple bounds (`<T:seq, to_str>`). What actually happens here is that vtables for the interface are passed as hidden parameters to the generic (much like type descriptors currently are). The call site knows the type of the argument, and can generate the right vtable pointer. When generics call each other, the vtables can be threaded through (again, in exactly the same way as tydescs).

An implementation that did not refer to an explicitly declared interface can still participate in dynamic dispatch. If it has been imported, and its signature matches (or is a superset of) the named interface, it will be considered an implementation of that interface.

Maybe we also want to allow interface implementations to be patched together out of multiple imported interfaces. I (Marijn) feel this is a bit too automatic, but I believe Niko would prefer that to be supported. We'll have to further discuss, or see in practice.

## Kinds as bounded type parameters

The `copy` and `send` kinds could be annotated just like other parameter bounds, removing a special case from our syntax. They behave very much like interfaces, except that they are not implemented by the user, but automatically derived by the compiler. We may want to look into automatic derivations for other interfaces (`to_str`, `compare`) as well in the future.

## Per-value vtables

The above does not help when you want to have a collection of generic values, each with its own vtable. For this, we can support explicit casting to vtable-wrapped types.

    circle(10f, "red") as drawable

Where `drawable` is an interface. This would copy value and the `drawable` vtable for its type into a reference counted box. You could then stuff a number of those into a collection, and call their `.draw()` methods through the boxed vtable.

    fn draw_all(ds: [drawable]) { for d in ds { d.draw(); } }

# Extensions

The below has not been properly worked out or discussed yet, but will undoubtedly come up.

## Qualified implementations

    iface hash { fn hash() -> uint }
    
    impl<T:hash> hash : [T] {
        fn hash() -> uint {
            let h = 0u;
            for elt in self { h += elt.hash(); }
            h
        }
    }

Not sure about that syntax, but what it is saying is 'if `T` has an implementation of `hash`, then this is an implementation of `hash` for `[T]`'. When creating the vtable for `[T]`, the vtable for `T` must first be looked up (in local scope), and then stored in the vtable for `[T]` somehow, so that it can be fed to the `hash` method.

## Alternate Syntax

I personally don't think it's a good idea to conflate the `I<T>` syntax used by generics with interfaces/typeclasses as well. What we're essentially creating here, for an interface `I`, is both a hidden typeclass `I`, as well as an existential type `I`. In this reasoning, the x:I syntax for "x has an implementation of I" makes perfect sense.

In addition, what if we want to define interfaces over parameterized types? These are extremely common and the syntax would become confusing with both meanings of `<_>` in use.

With these considerations, perhaps a better and more logical syntax for the above would be...

    iface F<T> : functor<T> { 
      fn map(f: block(T) -> S, thing:F<T>) -> F<S>; 
    }
    
    impl [T] : functor<T> {
      fn map(f: block(T) -> S, thing:F<T>) -> F<S>; 
        
      }
    }

The <T> on the interface itself is required. Multiple dispatch is achieved by simply separating the types on the left hand side of the `:` in the `iface`/`impl`. (Separating with commas is ambiguous with other grammar). For instance, we could implement static casts as follows:

    iface T S : castable {
      fn static_cast(T) -> S;
    }

    impl foo bar : castable {
      fn static_cast(f: foo) -> bar {
        // perform a conversion from foo to bar
      }
    }

C++ uses the confusing syntax static_cast<dest_type>(x), inferring the src_type from x. This is immensely confusing because it suggests a generic type, which in turn suggests it should be generic over ALL destination types. However this is untrue, and you may not be able to convert to a certain destination type.

In conclusion, I think the syntax we use for our interfaces should reflect their **existential** nature, and avoid conflation with **universal** (generic) syntax.

- Dylan

## Self type

Many interface specifications have to refer to the type of their `self` somehow. For example:

    iface send_to_mom { fn send(chan<self>) }

'Self types' are apparently a difficult problem in type theory. I'm don't have the background to say much about them, but I get the impression that this kind of interface system sidesteps a lot of their issues.

I don't think self types are a good idea, since we're not really talking about objects, and its somewhat unclear. Should `self` refer to the iface, or to the type being implemented? At least due to the first example on this page, this makes no sense to me. A `self` locally bound to the (existential) type of the interface itself would make sense. - Dylan

## Non-method-syntax methods

The dot-syntax is great for operations that obviously hang off a single value, but becomes awkward for binary or N-ary operations. For example a set union is traditionally done `set1.union(set2)` in OO languages. We could allow something like `union.(set1, set2)` to specify that `union` is to be looked up as a method.

However, this also requires implementations of `union` to spell out all their arguments, so there'd be some markup necessary in interface declarations to specify whether methods take an implicit self argument or not. Maybe by prefixing method names that expect implicit self arguments with a dot (`fn .len() -> uint`).

(This is for the future.)

## Operator overloading

We could allow operator names to appear as function names in interfaces, and include interfaces `cmp` and `num` in the core library with suitable implementations for the built-in types.

    iface num {
        fn +(self, self) -> self;
        fn -(self, self) -> self;
        /* etc */
    }

You could then implement operators for non-primitive types by defining (and importing) a suitable implementation.

    impl num : bignum {
        fn +(a: self, b: self) -> self { mp::add(a, b) }
        /* etc */
    }

