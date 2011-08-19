At the time of this writing (August 2011), Rust has a lightweight, structural object system with support for self-dispatch, method overriding, and object restriction.

## Self-dispatch

Here's a very simple example of an object where a method contains a self-call:

```
    obj cat() {
        fn ack() -> str {
            ret "ack";
        }
        fn meow() -> str {
            ret "meow";
        }
        fn zzz() -> str {
            ret self.meow();
        }
    }

    let shortcat = cat();

    assert (shortcat.zzz() == "meow");
```

Here we have an object definition, or _object item_ as it's called in Rust.  This definition causes there to be a constructor `cat`; we're calling that constructor to create an object called `shortcat`.  One of the methods on that object contains a self-call.

If the parser sees the token `self`, it assumes that whatever follows the dot is a call expression, and it parses the function part of that call expression into a special flavor of AST node (`expr_self_method`).  During typechecking, the crate context keeps a stack `obj_infos` to track the current object, so when it comes time to typecheck a self-call, we grab `self`'s type out of the current object type.  During translation, we again have a context that keeps track of the current object, if there is one, so we can look up the method's identifier on that object in much the same way that we would look up a field of a record.

### Limitations of `self`

Rust's notion of `self` is currently limited to support for self-calls like `self.meow()`.  The only thing you can put after the dot is an identifier, and `self` doesn't exist as a standalone, first-class entity; it's nothing more than a prefix to method calls.  You can't do things like return `self` from a method, nor can you write methods that explicitly take arguments of type `self`. 

## Object extension

An example of Rust's object extension syntax:

```
    let longcat = obj() {
        fn lol() -> str {
            ret "lol";
        }
        fn nyan() -> str {
            ret "nyan";
        }
        with shortcat
    };

    assert (longcat.zzz() == "meow");
```

Here we're extending `shortcat` (see `with shortcat` clause) with two new methods to create an object called `longcat`.  The expression being assigned to `longcat` is what Rust calls an _anonymous object expression_.  It's not a definition as for `shortcat`, rather, it's an expression that can appear in any expression context and evaluates to an object.  No constructor is called; instead, the object is created "inline".

`longcat` is a bona fide object with five entries in its vtable, appearing in alphabetical order: `ack`, `lol`, `meow`, `nyan`, `zzz`.  Rather than "copying forward" stuff from the object being extended, Rust puts a wrapper around it: `lol` and `nyan` are "normal" vtable entries, and the others are "forwarding functions" that, roughly speaking, forward us along to the appropriate vtable entries from `shortcat`.

### Backwarding

When we call `longcat.zzz()`, we're forwarded along to `shortcat`'s vtable entry for `zzz`, which contains the compiled code for `ret self.meow()`.  However, `shortcat`'s vtable (which contains three entries, `ack`, `meow`, and `zzz`, in that order) was created under the assumption that `self` is `shortcat`.  The call to `self.meow()` therefore compiles to the equivalent of "do whatever the (zero-indexed) slot 1 in my vtable says to do".  At this point, `self` is supposed to be `longcat`, but if we simply use `longcat` for `self`, since the zero-indexed slot 1 of `longcat` will be the wrong method -- `lol` instead of `meow`.

We need whatever we use for `self` inside `shortcat` to have the same type as `shortcat` does.  By "same type", we mean that its vtable must have the same number of methods, with the same names, in the same order, and having the same types as the original.  We accomplish this by adding another level of indirection.  When we create the forwarding functions, we replace `self` with a vtable that contains three "_backwarding_" unctions, one for each of the original three methods `ack`, `meow`, and `zzz`, which point to the corresponding entries in `longcat`'s vtable.  From there, of course, they'll forward right back to `shortcat`'s vtable -- unless they're being overridden.
 
## Overriding

Rust allows methods in an extended object to override methods on the original:

```
    let longercat = obj() {
        fn meow() -> str {
            ret "zzz";
        }
        with shortcat
    };

    assert (longercat.zzz() == "zzz");
```

Here, the call `longercat.zzz()` will forward us to the `zzz` method on `shortcat`.  The call `self.meow()` will use the backwarding vtable provided for `shortcat`, which will point us to `longercat.meow()`, which returns "zzz", as we expect.  So we get the overriding behavior "for free" with the backwarding/forwarding architecture that makes object extension possible.

## Multi-level extension

Objects can be extended to arbitrary depth.  At this point, we could write:

```
    let evenlongercat = obj() {
        fn meow() -> str {
            ret "zzzzzz";
        }
        with longercat
    };

    // Tests two-level forwarding/backwarding + self-call + override.
    assert (evenlongercat.zzz() == "zzzzzz");
```

This behavior is implemented using, once again, a stack that keeps track of what vtable should be used for `self`.  This stack is pushed and popped each time a forwarding or backwarding call is made, respectively, and is threaded through the run-time stack using space allocated in the forwarding and backwarding functions' frames.

## Self-types

The notion of self-type, or "the type of the current object", is a type-theoretic approach to reasoning about the semantics of self-dispatch (and expressions involving "self" in general).  Object extension and method overriding are the situations that make self-types most useful and interesting, since, as illustrated above, those are the situations that change the meaning of "self" at run-time.  Although the Rust object system doesn't use self-types explicitly, it's possible that what Rust has would fit into a theory of objects that uses self-types.

### Some relevant literature

Bono et al. [1] write: "an appropriate type for self would allow to specialize automatically those inherited/overridden methods that either return the host object, or have some parameters of the same
type as the host object (binary methods)" -- that is, return `self` and take arguments of
type `self`  Fisher et al. [2] is another example of a system with self-types, and like Bono et al., deals with nesting.

Abadi and Cardelli's "Theory of Objects" tutorial from OOPSLA '96 [3] suggests that the simplest object calculus with self types is ςOb ("sigma-ob"), (Slide 42 (p. 11 of the PDF).  Of the features they consider, sigma-ob has only objects, object types, subtyping, and self types.  The sigma-ob paper [4]  explains why using ordinary recursive types to try to implement self doesn't work in the presence of method override.  They offer two fixes for this problem.  The fix laid out in [4] uses a "Self quantifier"; the other fix is "the standard solution", which is apparently what Modula-3 did, and is laid out in [5]; its disadvantage is that it  "sacrifices static typing information which must be recovered dynamically, thus abandoning the static typing of subsumption".

[1] Type Inference for Nested Self Types.  Viviana Bono, Jerzy Tiuryn,
and Pawel Urzyczyn.
http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.106.7277&rep=rep1&type=pdf

[2] A Lambda Calculus of Objects and Method Specialization.  Kathleen
Fisher, Furio Honsell, and John C. Mitchell.
http://citeseer.ist.psu.edu/viewdoc/download?doi=10.1.1.48.5828&rep=rep1&type=pdf

[3] A Theory of Objects (OOPSLA Tutorial).  Martin Abadi and Luca Cardelli.
http://lucacardelli.name/Talks/1996-10%20A%20Theory%20of%20Objects%20(OOPSLA%20Tutorial).pdf

[4] A Theory of Primitive Objects: Second-Order Systems.  Martin Abadi and Luca Cardelli.
http://lucacardelli.name/Papers/PrimObj2ndOrder.A4.pdf

[5] A theory of primitive objects: Untyped and first-order systems. Martín Abadi and Luca Cardelli. 
http://lucacardelli.name/Papers/PrimObj1stOrder.A4.pdf