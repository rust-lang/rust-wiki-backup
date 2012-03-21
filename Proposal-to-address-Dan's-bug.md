# A solution to Dan's bug based on immutability

## Dan's bug

The bug we are trying to address is when an alt creates a reference
into memory whose type may change through mutation or which may be
deallocated through mutation.

Example #1 (enum variants):

    type T = { mutable x: ast::expr };
    fn foo(t: T) {
        alt t.x {
            expr_binary(op, l, r) {
                t.x = expr_unary(...);
                // op, l, and r are now invalidated
            }
        }
    }

Example #2 (pointers):

    type T = { mutable x: @ast::expr };
    fn foo(t: T) {
        alt t.x {
            @expr_binary(op, l, r) {
                t.x = @expr_unary(...);
                // op, l, and r *may* have been freed now
            }
        }
    }

Today we prevent these via approximate, type-based alias analysis.

## Summary

- Only allow variant patterns to be matched against immutable memory
- Only allow dereferencing (auto or otherwise) in patterns against
  immutable memory.
- Fix a type hole in current system where immutability does not in fact
  mean "won't be modified"
- In process, introduce a bound `assign` which refines `copy` to indicate
  types that can safely be overwritten

## The solution

The basic solution is to prohibit dangerous patterns from being
matched against mutable memory.  A dangerous pattern is either:

- a variant pattern;
- a box pattern `@P` or unique pattern `~P` (both of which perform a
  dereference).

By design, our type system distinguishes potentially mut memory from
other memory (or it's supposed to, see details below).  Therefore, we
simply modify pattern matching so that matching a variant against mut
memory is a static error.  We also allow patterns to be preceded by a
copy keyword, in which case the match occurs not against the structure
itself but instead against a temporary copy of the structure's
contents onto the stack.  A copy of an enum variant will guarantee
that it remains immutable; a copy of an `@` or `~` pointer will
guarnatee that the value being matched against remains live.

> Example:
> 
>     type T = {
>         f: ast::expr,
>         mut g: ast::expr,
>         mut h: @ast::expr,
>     };
>     fn foo(t: T) {
>         alt t.f {
>             // OK: t.f is immut
>             expr_binary(...) { }
>             ...
>         }
>         
>         alt t.g {
>             // ERROR: t.g is mut
>             expr_binary(...) { }
>             
>             // OK: pattern is copied.
>             // Alternatively, you could
>             // do `match copy t.g`.
>             // This inline copy form is provided
>             // to support nested patterns.
>             // Perhaps it is not needed.
>             copy expr_binary(...) { }
>             
>             ...
>         }
>         
>         alt t.h {
>             // OK: t.h is mut, but what's
>             // being matched here is the autoderef'd
>             // box, which is immut.
>             expr_binary(...) { }
>             ...
>         }
>     }

## Defining mut memory

This entire concept is premised on the ability of the type system to
detect "mutable" memory.  The current system of references is not able
to do that very well, but a region-based type system will be able to
do so.  

The basic requirement is that we must be able to determine whether
any given bit of reachable memory is mutable or not.  We rely on a few
recent additions to make this possible:

- local variables that are tagged as mutable: variables not tagged as `mut`
  can never be updated, so we know whether that portion of the stack is
  mutable or not.
- region pointers in place of references: a region type like `&T` can
  only point at immutable memory; similarly, `&mut` can only point at
  mutable memory.  `&const` can point at either.  Only the first can
  be considered immutable.

Based on this, a given lvalue L is potentially mutable under the following
conditions (here, we ignore autoderef, which is simply a pre-expansion step):

- L = L'.f where f is f declared mutable
- L = L'.f where L' is potentially mutable
- L = x where x is a local variable declared mutable
- L = x where x is a mutable upvar captured by reference (that is, in an fn&)
- L = *L' where L' has type &mut or &const
- L = *L' where L' has type @mut or @const
- L = *L' where L' has type ~mut or ~const
- L = L'[_] where L' has type [mut _] or [const _]

## Appendix A: Type system hole

The current type system using references is unsound because it cannot
determine, based on the types alone, whether a given lvalue is
mutable.  Consider this example, which creates an immutable box but
then manages to mutate it:

```
type T = { f: @const int };

fn foo(&t: T, v: @const int) {
    t = {f:v};
}

fn main() {
    let h = @3; // note: h is immutable
    let g = @mutable {f: @mutable 4};
    #error["h=%? g=%?", h, g]; // prints "h=@3 g=@(@4)"
    foo(*g, h);
    #error["h=%? g=%?", h, g]; // prints "h=@4 g=@(@4)"
    *g.f = 5;
    #error["h=%? g=%?", h, g]; // prints "h=@5 g=@(@5)"
}
```

In a new region-like universe, this program would be written like so:

```
type T = { f: @const int };

fn foo(t: &mut T, v: @const int) {
    t = {f:v};
}

fn main() {
    let h = @3; // note: h is immutable
    let g = @mutable {f: @mutable 4};
    foo(g, h); // ERROR
    *g.f = 5;
}
```

The line marked `ERROR` indicates where the type check would fail: the
type of `g` is `@mutable {f: @mutable int}`.  This can be implicitly
coerced to `&mutable {f: @mutable int}`, but it is not a subtype of
`&mutable {f: @const int}`.  In the old system, the mutability that
was derived from the "by-mutable-reference" kind of the parameter `t`
was invisible to the type system, and hence the type system applied a
covariant rule, leading to the hole.

