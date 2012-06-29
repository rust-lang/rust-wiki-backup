# Unifying traits and interfaces

There are three parts to this proposal:

  * Adding default impls to ifaces
  * Allowing iface composability
  * Instance coherence: only one impl per iface/type pair

Then, rename `iface` to `trait` and that's it!

## Adding default impls to ifaces

### An example

In the `middle::typeck::infer` module (henceforth `infer`), there's a
`combine` interface, and implementations of that interface for the
three "type combiners" `lub`, `sub`, and `glb`.  All three `impl`s are
required to implement all of the methods in the `combine` interface,
even though some of the implementations are identical in two (or in
all three!) of the type combiners.  Right now, `infer` deals with this
by defining an out-of-line method for each method for which there are
multiple identical implementations, and having all the different
implementations call the out-of-line-method.

For example, here's what it looks like for the `modes` method.  In
fact, there are _nine_ methods in `infer` that are this way -- `modes`
is just a representative example.  The `infcx` method is also
identical in all three, but since all it does is return `*self`, it's
just identically implemented in all three.

```
iface combine {
    fn infcx() -> infer_ctxt;
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode>;
    ...
}

impl of combine for sub {
    fn infcx() -> infer_ctxt { *self }
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        super_modes(self, a, b)
    }
    ...
}

impl of combine for sub {
    fn infcx() -> infer_ctxt { *self }
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        super_modes(self, a, b)
    }
    ...
}

impl of combine for glb {
    fn infcx() -> infer_ctxt { *self }
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        super_modes(self, a, b)
    }
    ...
}

// Out-of-line method
fn super_modes<C:combine>(
    self: C, a: ast::mode, b: ast::mode)
    -> cres<ast::mode> {

    let tcx = self.infcx().tcx;
    ty::unify_mode(tcx, a, b)
}
```

Under this proposal, we could put the default implementation in the
interface, so, instead of all of the above, we could just write:

```
trait combine {
    fn infcx() -> infer_ctxt { *self }
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        let tcx = self.infcx().tcx;
    	ty::unify_mode(tcx, a, b)
    }
    ...
}

impl of combine for sub {
    ... // only methods for which the default impl isn't enough
}

impl of combine for sub {
    ... // only methods for which the default impl isn't enough
}

impl of combine for glb {
    ... // only methods for which the default impl isn't enough
}
```

The only other thing that changed in this code was that the keyword
`iface` changed to `trait`.

## Allowing iface composability

Traits, as they appear in the literature, have a set of _provided_
methods, implementing the behavior that a trait provides, and a
(possibly empty) set of _required_ methods that the provided methods
can be written in terms of.  For the required methods, only the names
and types are specified, not the implementation.  That suggests that
in Rust, a trait's set of required methods could be specified using an
iface.  But if traits themselves _are_ ifaces, then that means that
ifaces can require ifaces.  This goes along with the idea that traits/ifaces
should be _composable_ and _order-independent_: a trait C can extend traits A
and B (in either order).

In today's Rust, a class or a type can implement interfaces A and B
(in either order).  For instance, in today's Rust you can write:

```
iface add { fn plus(n: int) -> int; }
iface subtract { fn minus(n: int) -> int; }

impl of add for int {
    fn plus(n: int) -> int { self + n }
}

impl of subtract for int {
    fn minus(n: int) -> int { self - n }
}

// int implements both add and subtract; order doesn't matter
fn main() { assert 3.plus(1) == 5.minus(1); }

```

But interfaces themselves aren't composable: we can't currently define
an interface as the composition of more than one interface.  Under
this proposal, it would be possible to write:

```
iface arithmetic: add, subtract {
    ... // more methods here, if we want
}
```

The `add, subtract` part of the signature amounts to a set of required
methods, in terms of which additional methods can be written.

Then, adding the ability to put default impls in ifaces (and changing
the keyword from `iface` to `trait`), we get:

```
trait add { 
    fn plus(n: int) -> int { self + n }
}

trait subtract {
    fn minus(n: int) -> int { self - n }
}

trait arithmetic: add, subtract {
    ... // more methods here, if we want
}

impl int: arithmetic; // or something like this

fn main() { assert 3.plus(1) == 5.minus(1); }

```
One place that could benefit from this so-called 'interface inheritance' is called out by a FIXME for issue #2616 in `core::num`.  Although I have to think about it some more, I _think_ we could clean up duplicated code between `core/int_template.rs` and `core/uint_template.rs` with this kind of strategy.

### Conflict resolution

Traditional traits do some cool conflict resolution stuff when the
traits being combined have methods with the same name, and we might
want to do that eventually, but we can punt for now and just do what
Rust already does if a type implements multiple interfaces that define
a method with the same name, that is, raise a compile-time "multiple
applicable methods in scope" error.

## Instance coherence: "one impl of an iface per type"

Instance coherence means that there can be at most one implementation
of a trait for a particular type.  Two rules, originally suggested by
pcwalton, are sufficient to enforce this:
  
  1. The default implementations defined within a trait must be
     non-overlapping (i.e., there can’t be any types that match
     multiple implementations).

  2. A class can’t derive from a trait that defines a default
     implementation that might itself match an instance of that class.

It offers coherence because there can be only one implementation of an
trait for each type. For the implementations provided within the trait
itself, we can check that they’re nonoverlapping. For the
implementations defined within a class, we can check to ensure that
the implementations defined in any trait that the class derives from
don’t overlap with the implementations that the interface itself
defined. Either way, the checks involved are simple and ensure that we
meet the criterion for coherence.



TODO.