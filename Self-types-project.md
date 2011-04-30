Bono et al. [1] say, "The gain of introducing an
appropriate type for self is evident when a form of inheritance is
present, either a class-based one (via class hierarchies), or an
object-based one (via method addition/override)."  We don't have classes or a class hierarchy in Rust, and our documentation claims that we do not have "any concept of inheritance".  The same documentation says
that "method overriding and object restriction are performed explicitly on object values", which sounds like
what Bono et al. would call a form of inheritance, albeit an object-based one.

However, method overriding and object restriction are not actually implemented yet; neither is method addition.  We'd like to implement method overriding in a way that works more or less analogously to our existing functional record update, which is available in Rust via the `with` syntax (although we don't have similar operations on records that are analogous to method addition or restriction, as in actually adding fields to a record or removing them).

For objects, rather than "copying forward" stuff from the extended-or-overridden object, we would wrap it in an adaptor.  (Graydon: "All delegation is wrapping.")  We'd have an expression form that would look something like:

    obj { fn foo() { ... new method ... } with other_obj }

where `other_obj` is an already-defined object.  So step one for implementing this would be "anonymous object expressions", and then object expressions that wrap inner objects.  This is something like prototype-based OO, although according to Graydon, we're "flattening the prototype chain".

Again, the reason all this is relevant is because, as Bono et al. say, having self-types is particularly useful in a situation that has some form of inheritance (even if it's "only" prototype-style inheritance).  As a concrete example, say you have the following:

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

Now, my_b is an object, so it's going to be a pair of a vtbl pointer and
a body pointer:

     my_b: [vtbl* | body*]

my_b's vtbl has entries for foo, bar, and baz, whereas my_a's vtbl has
only foo and bar.  my_b's 3-entry vtbl consists of 2 forwarding
functions and 1 real method.

my_b's body just cotains the pair a: [ a_vtbl | a_body ], wrapped up
with any additional fields that my_b added.  None were added, so my_b
is just the wrapped inner object.

When you call my_b.foo(), it calls self.a.foo(), essentially, and that
latter call passes the my_a object as 'self' to a.foo().  This is
"wrapping the inner object to appear like the outer object", also
known as wrapping, period.

The less-expected direction, wrapping the outer object to appear like
the inner, happens when you *override*, not just extend.  Suppose we
have an object that extends my_a, overriding foo():

     auto my_c = obj { fn foo() -> int { return 3; } with my_a };

Now, if we call my_c.bar(), it calls self.a.bar(), which passes the
my_c object as 'self' to a.bar(), which calls looks up the foo()
method of self, so it should hit my_c.foo().  I *think* I have this correct -- I hope -- but I'm not totally sure.

Bono et al. go on to say: "In fact, an appropriate type for self would allow
to specialize automatically those inherited/overridden methods that
either return the host object, or have some parameters of the same
type as the host object (binary methods)."  These sound like the
things we want to be able to do: return self and take arguments of
type self.  I hadn't thought about the problem as one having to do
with inheritance.

In the second section of Bono et al.:

"It might seem natural to identify objects with recursive records ,so
their types with recursive types, but this is misleading." Curious to
see why.

"This paper might be seen as a first step towards introducing
selftypes in real programming languages as an alternative to
typecasts."  Great!  That's what we want!

M <= i extracts the i-th method from object M.  Operational semantics:

pro s<M_1, M_2> <= i --> M_i[pro s<M1, M2>/s]

that is, M_i but with pro s<M1, M2> in place of occurrences of the
bound variable s.  This smacks of recursive types.

"Another reason why we do not want to use recursive types is that we
certainly want to distinguish between pro s<3, pro s<2, s>> and pro
s<2, s>."  Wait, this just sounds like we're just saying that we want
iso-recursive types rather than equi-recursive types.

Fisher et al. [2] is another example of a system with self-types, and like Bono et al., deals with nesting.  I like the notation better and the paper is interesting but I'm not seeing how it is immediately relevant to the problem of assigning a type to `self`.

Abadi and Cardelli's "Theory of Objects" tutorial from OOPSLA '96 [3]
looks kind of promising, though!  Slide 42 (p. 11 of the PDF) lists a bunch of
object calculi with various features.  it appears from this that the
simplest object calculus with self types is ςOb ("sigma-ob").  Of the features they're considering, sigma-ob
has only objects, object types ,subtyping, and self types.  Notably, it doesn't have recursive types.

(Interestingly, two of the systems that they list as having recursive types also have an open (not-filled-in) circle in the "self-types" spot -- I don't know what it means, but maybe it means something like "can encode self-types".  What else do those systems have that would enable such an encoding?  Subtyping?  Variance?  (What's variance?)

And then there's the sigma-ob paper [4], which looks promising...

[1] Type Inference for Nested Self Types.  Viviana Bono, Jerzy Tiuryn,
and Pawel Urzyczyn.
http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.106.7277&rep=rep1&type=pdf

[2] A Lambda Calculus of Objects and Method Specialization.  Kathleen
Fisher, Furio Honsell, and John C. Mitchell.
http://citeseer.ist.psu.edu/viewdoc/download?doi=10.1.1.48.5828&rep=rep1&type=pdf

[3] A Theory of OBjects (OOPSLA Tutorial).  Martin Abadi and Luca Cardelli.
http://lucacardelli.name/Talks/1996-10%20A%20Theory%20of%20Objects%20(OOPSLA%20Tutorial).pdf

[4] A Theory of Primitive Objects: Second-Order Systems.  Martin Abadi and Luca Cardelli.
http://lucacardelli.name/Papers/PrimObj2ndOrder.A4.pdf