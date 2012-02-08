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

We already have a definition for when an lvalue is mutable.  The
algorithm is given in `mut.rs` and we can mostly reuse it, though we
will have to be careful around `&&` references, as these may in fact
point to mutable memory. However, there is
a hole in our type system (see Appendix A) which actually makes this algorithm unsound
(and, indeed, the type system itself, which treats supposedly
immutable fields covariantly, and probably alias analysis too).  We
must therefore close this hole first (we should do this anyway).

The problem is due to types like `@mut T` or `[mut T]` where `T` is an
aggregate type. Let's focus on `@mut T`, `[mut T]` is analogous. Here
is an example showing the problem:

    type T = { f: int, g: int }; // note: f and g are immutable
    fn foo() {
        let x = @mut {f: 3, g: 5};
        let y = @mut {f: 4, g: 6};
        
        // Here x.f == 3

        *x = y;
        
        // Here x.f == 4, but wait, wasn't it immutable?
    }
    
As this example demonstrates, an immutable field in our system isn't
really immutable: it's only immutable if contained in an immutable
context.  But we *treat* such fields as immutable in numerous places
(as, indeed, I think we should).

## The `assign` type kind

To close the type hole described in the previous section, we introduce
an `assign` type kind that indicates whether a type is assignable.
Most copyable types are also assignable.  The only exceptions are:

- enums
- records with immutable fields or fields of non-assignable type
- tuples

An assignment `l=r` is permitted only if the type of `l` is
assignable.  Similarly, a mutable field (resp. box, vector) can only
be created if the type is assignable.

## Appendix A: Type system hole

To show why the current type system is unsound, consider this example, which creates an immutable box but then manages to mutate it:

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