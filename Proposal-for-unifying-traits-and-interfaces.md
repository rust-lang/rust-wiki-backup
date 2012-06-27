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
by defining an out-of-line method `super_foo` for each method `foo`
for which there are multiple identical implementations.

For example, here's what it looks like for the `modes` method.  In
fact, there are _nine_ methods in `infer` that are this way -- `modes`
is just a representative example.

```
iface combine {
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode>;
    ...
}

impl of combine for sub {
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        super_modes(self, a, b)
    }
    ...
}

impl of combine for sub {
    ...
    fn modes(a: ast::mode, b: ast::mode) -> cres<ast::mode> {
        super_modes(self, a, b)
    }
    ...
}

impl of combine for glb {
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

## Instance coherence

One impl of an iface per type

TODO.