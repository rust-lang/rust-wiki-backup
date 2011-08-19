At the time of this writing, Rust has a lightweight, structural object system with support for self-dispatch, method overriding, and object restriction.

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

Here we're extending `shortcat` (see `with shortcat` clause) with two new methods to create an object called `longcat`.  The expression being assigned to `longcat` is what Rust calls an anonymous object expression.  It's not a definition as for `shortcat`, rather, it's an expression that can appear in any expression context and evaluates to an object.  No constructor is called; instead, the object is created "inline".

`longcat` is a bona fide object with five entries in its vtable.  Of those, `lol` and `nyan` are "normal" entries, and the others are "forwarding functions" that, roughly speaking, forward us along to the appropriate vtable entries from `shortcat`.  (Vtable entries appear in alphabetical order: `ack`, `lol`, `meow`, `nyan`, `zzz`.)  Rather than "copying forward" stuff from the object being extended, Rust puts a wrapper around it.

### Backwarding

When we call `longcat.zzz()`, we're forwarded along to `shortcat`'s vtable entry for `zzz`, which contains the compiled code for `ret self.meow()`.  However, `shortcat`'s vtable (which contains three entries, `ack`, `meow`, and `zzz`, in that order) was created under the assumption that `self` is `shortcat`.  The 





The reason all this is relevant to self-types is that having self-types is particularly useful in a situation that has some form of inheritance (even if it's "only" prototype-style inheritance).  As a concrete example, say you have the following:

     obj a() {
         fn foo() -> int {
                ret 2;
         }
         fn bar() -> int {
                ret self.foo();
         }
     }

     auto my_a = a();
     auto my_b = obj { fn baz() -> int { ret self.foo() } with my_a };

Here we're making up syntax to extend the `my_a` object with an additional method `baz`, creating an object `my_b`.  Since it's an object, `my_b` is a pair of a vtable pointer and a body pointer:

     my_b: [vtbl* | body*]

`my_b`'s vtable has entries for `foo`, `bar`, and `baz`, whereas `my_a`'s vtable has
only `foo` and `bar`.  `my_b`'s 3-entry vtable consists of two forwarding functions and one real method.

`my_b`'s body just contains the pair `a: [ a_vtable | a_body ]`, wrapped up
with any additional fields that `my_b` added.  None were added, so `my_b`
is just the wrapped inner object.

When we call `my_b.foo()`, it calls `self.a.foo()`, essentially, and that
latter call passes the `my_a` object as 'self' to `a.foo()`.  This is
"wrapping the inner object to appear like the outer object".

The less-expected direction, wrapping the outer object to appear like
the inner, happens when we *override* a method in `my_a`, rather than just extend it.  Suppose we
have an object that extends `my_a`, overriding `foo`:

     auto my_c = obj { fn foo() -> int { ret 3; } with my_a };

Now, if we call `my_c.bar()`, it calls `self.a.bar()`, which passes the
my_c object as 'self' to `a.bar()`, which calls looks up the `foo()`
method of self, so it should hit `my_c.foo()` and return 3 rather than 2.

## Hopefully-relevant literature

Bono et al. [1] say: "an appropriate type for self would allow
to specialize automatically those inherited/overridden methods that
either return the host object, or have some parameters of the same
type as the host object (binary methods)."  These sound like the
things we want to be able to do: return self and take arguments of
type self -- I hadn't thought about the problem as one having to do
with inheritance.

In the second section of [1]:

  * "It might seem natural to identify objects with recursive records, 
their types with recursive types, but this is misleading." Curious to
see why.

  * "This paper might be seen as a first step towards introducing selftypes in real programming languages as an alternative to typecasts."  Great!  That's what we want!

  * The notation `M <= i` extracts the `i`-th method from object `M`.  Operational semantics: `pro s<M_1, M_2> <= i --> M_i[pro s<M1, M2>/s]`.  That is, extracting the `i`-th method gives us `M_i` but with `pro s<M1, M2>` in place of occurrences of the bound variable `s`.  This smacks of recursive types.

  * "Another reason why we do not want to use recursive types is that we certainly want to distinguish between `pro s<3, pro s<2, s>>` and `pro s<2, s>`."  To me this just sounds like we're saying that we want
iso-recursive types rather than equi-recursive types.

Fisher et al. [2] is another example of a system with self-types, and like Bono et al., deals with nesting.  I like the notation better, and the ideas are nice (lambda-tastic object encodings), but I don't see immediate relevance to the problem of assigning a type to `self` -- although I'm probably missing something.

Abadi and Cardelli's "Theory of Objects" tutorial from OOPSLA '96 [3] looks kind of promising.  Slide 42 (p. 11 of the PDF) lists a bunch of object calculi with various features.  it appears from this that the
simplest object calculus with self types is ςOb ("sigma-ob").  Of the features they're considering, sigma-ob
has only objects, object types, subtyping, and self types.  Notably, it doesn't have recursive types.  (Interestingly, two of the systems that they list as having recursive types also have an open (not-filled-in) circle in the "self-types" spot -- I don't know what it means, but maybe it means something like "can encode self-types".  What else do those systems have that would enable such an encoding?  Subtyping?  Variance?  (What's variance?)

The sigma-ob paper [4], which I'm reading now, is looking very promising.  I haven't got my head around it yet completely, but it explains why, if we used regular recursive types to try and implement self, that would just be *wrong* in the presence of method override.  They offer two fixes for this problem.  The fix laid out in [4] uses a second-order quantifier sigma (different from the sigma appearing at the term level!) that I don't understand yet.  The other fix is "the standard solution", which is apparently what Modula-3 did, and is laid out in [5].  I'm going to look into the latter next.  Apparently the disadvantage of that solution is that it  "sacrifices static typing information which must be recorvered dynamically, thus abandoning the static typing of subsumption".  I'm not entirely sure what that means yet.

Also, suggested by mccr8 ("self types sound a little like the typing issues involved with typing functions with explicit self parameters, as arise in the typed compilation of OO languages"):

 * http://research.microsoft.com/apps/pubs/default.aspx?id=59934  
   A simple typed intermediate language for object-oriented languages.  Juan Chen and David Tarditi.  POPL 2005. ("focuses just on the encoding technique" -- mccr8)

 * http://contrapunctus.net/league/research/papers/pip03.pdf
   Precision in practice: A type-preserving Java compiler.  C. League, Z. Shao, and V. Trifonov.  CC 2003. 

---

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







