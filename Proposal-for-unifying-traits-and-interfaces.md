# Unifying traits and interfaces

## A real example of how we could use traits

In middle::typeck::infer (henceforth "infer"), there's a `combine`
interface, and impls of that interface for the three "type combiners"
`lub`, `sub`, and `glb`.  Each of these impls is required to implement
all of the methods in the `combine` interface, even though some of the
implementations are identical in two (or in all three!) of the .  The
way that infer deals with this now is that for each method `foo` that
has identical implementations, there's an out-of-line method
`super_foo`.

For example, here's how it works for the `modes` method.  This is all
code taken directly from infer.  In fact, there are **nine** methods
in infer that are this way.  I'm just picking `modes` as a
representative example.

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

With traits, we could put the default implementation in the interface,
so, instead of all of the above, we could just write:

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