# Unifying traits and interfaces

There are three parts to this proposal:

  * Adding default impls to ifaces
  * Allowing iface inheritance
  * Instance coherence

### Adding default impls to ifaces: a real example

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

    fn args(a: ty::arg, b: ty::arg) -> cres<ty::arg> {
        super_args(self, a, b)
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
The only other thing that we changed in this code was to change the keyword `iface` to `trait`.

## Allowing iface inheritance

Traits and interfaces have the common characteristic that they're
_composable_ and _order-independent_: a class can derive from traits A
and B (in either order), and a class can implement interfaces A and B.

TODO.

## Instance coherence

TODO.