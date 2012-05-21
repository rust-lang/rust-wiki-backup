## What we want

This subtyping relationship should hold:

    fn<&x>(&x.T) -> T' <: fn<&y>(&y.S) -> S'

Here, `x` and `y` are two region variables being introduced for the first time, and it's irrelevant what the types `T`, `T'`, `S`, and `S'` are.

This relationship:

    fn<&x>(&x.T) -> T' <: fn(&y.S) -> S'

should also hold, because in this case, we ought to be able to use
functions that have the subtype in any situation where we can use
function that have the supertype (since `y` is a free variable
representing a particular region, and `x` is a bound variable
representing *all* regions -- a set that includes `y`).

But this relationship:

    fn(&x.T) -> T' <: fn<&y>(&y.S) -> S'

(where `x` is free and `y` is bound) should *not* hold, because here the
alleged subtype is a function that only accepts a pointer with the
lifetime `x`, and the alleged supertype accepts a pointer with any
lifetime.  So we can't use the subtype in any context that expects the
supertype (because the context might want to pass in any old pointer).

## How to enforce it

We can enforce the above subtype relationships using the following algorithm:

  * Instantiate each bound region in the subtype with a fresh region
    variable.
  * Instantiate each bound region in the supertype with a fresh
    concrete region.
  * Now see if the subtyping relationship holds (performing
    unification and so on).

Using this algorithm on the first example above, we come up with

    fn(&X.T) -> T' <: fn(&y'.S) -> S'

where `X` is a fresh region variable and `y'` is a fresh concrete region.
Then we check if that holds.  It ought to, because `X` and `y'` should
unify.

For the second example above, we come up with

    fn(&X.T) -> T' <: fn(&y.S) -> S'

where `X` is a fresh region variable.  Then we check if that holds.  It
ought to, because `X` and `y` should unify.  (They unify even
though `y` is a "real" concrete region, because there's no reason why it
shouldn't unify with a fresh region variable!)

In fact, as an optimization, in the first example we don't actually have to create a fresh concrete region (that is, `y'`)
in the supertype.  That seems to make sense, because then it would
just work like the second example.  Niko pointed out that this trick is
"skolemization".  In trying to figure out what this has to do with
skolemization as I understand it (basically, getting rid of
existential quantifiers and creating new terms to replace them -- see a note about this below), it
occurred to me that maybe there's an implicit *existential* binder
around the types that have free region variables above.  So, if y is
one such, then maybe this is the optimization discussed [here](https://en.wikipedia.org/wiki/Skolem_normal_form#Uses_of_Skolemization),
where "only variables that are free in the formula are placed in the
skolem term".  I'm not really sure yet.

Anyway, in the third example above, there are no bound regions in the
subtype, and there's a bound region in the supertype.  So we get:

    fn(&x.T) -> T' <: fn(&y'.S) -> S'

where `y'` is a fresh concrete region.  Unification fails, because `x` and
`y'` are just different concrete regions and can't unify with each
other.

## Skolemization

This section is more or less a gloss of the [Wikipedia article](https://en.wikipedia.org/wiki/Skolem_normal_form).

Skolemization is the process of getting rid of existential quantifiers and creating
new terms to replace them.  In the simplest case (if there aren't any
universal quantifiers to contend with), `exists x. P(x)` can change to
simply `P(c)` where `c` is a new constant.  This makes sense -- `c` is
just the `x` that we know exists.

If there are some universal quantifiers around the thing -- say,
`forall x. exists y. forall z. P(x, y, z)` -- then skolemization gives
us `forall x. forall z. P(x, f(x), z)`, where `f` is a new function that
takes `x` as its argument.  `f` has to take `x` as an argument because `x` was
bound by the universal quantifier that was outside the existential
quantifier that was removed.  This technique generalizes to any
nesting of universal and existential quantifiers.