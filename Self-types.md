My project for the summer is to design and implement self-types (that is, "the type of the current object", as found in, for example, Scala) for the Rust type system.  The only notion of "self" that I've implemented so far is support for self-calls like `self.foo()`.  `self` still doesn't exist as a standalone, first-class entity; right now, it's nothing more than a magic prefix to method calls, and you can't do things like return `self`, nor can you write functions that take arguments of type `self`. For that, we need self-types.

## Method overriding/addition/restriction

The Rust documentation says that "method overriding and object restriction are performed explicitly on object values", but method overriding and object restriction are not actually implemented yet; neither is method addition.  We'd like to implement method overriding in a way that works more or less analogously to our existing functional record update, which is available in Rust via the `with` syntax.  (We don't have, and probably don't need, similar operations on records that are analogous to method addition or restriction, as in actually adding fields to a record or removing them).

For objects, rather than "copying forward" stuff from the extended-or-overridden object, we would wrap it in an adaptor.  (Graydon: "All delegation is wrapping.")  We'd have an expression form that would look something like `obj { fn foo() { ... new method ... } with other_obj }`, where `other_obj` is an already-defined object.  So, step one for implementing this would be "anonymous object expressions", and then object expressions that wrap inner objects.  This is kind of like prototype-based OO, although according to Graydon, we're "flattening the prototype chain".

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
     auto my_b = obj { fn baz() -> int { return self.foo() } with my_a };

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
the inner, happens when we *override*, a method in `my_a`, rather than just extend it.  Suppose we
have an object that extends `my_a`, overriding `foo`:

     auto my_c = obj { fn foo() -> int { return 3; } with my_a };

Now, if we call `my_c.bar()`, it calls `self.a.bar()`, which passes the
my_c object as 'self' to `a.bar()`, which calls looks up the `foo()`
method of self, so it should hit `my_c.foo()`.  (I *think* I have this correct -- I hope -- but I'm not totally sure.)

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