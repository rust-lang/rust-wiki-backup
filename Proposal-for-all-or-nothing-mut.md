# All or nothing mutability: a strawman

This is an alternative way to specify mutability in the Rust type
system. The goal is to retain the ability to have mutable lvalues of
record type while also retaining the ability for the type system to
know statically when memory is immutable or not.

This is only a strawman proposal.  It may or may not make sense.  I
just wanted to explore some of the alternate designs that are
available.

## Types

Types would be defined more-or-less the way they are today:

    Ty = { Id_1: Ty_1, ..., Id_N: Ty_N }  // Record types.
       | Id                               // Nominal types.
       | (Ty_1, ..., Ty_N)                // Tuple types.
       | ~Ty                              // Unique ptrs.
       | @ M Ty                           // Boxed values.
       | & M Ty                           // References (safe ptrs)
       | [M Ty]                           // Vectors.
       | ...

    M =        // Default: immutable
      | mut    // Mutable
      | const  // Read-only

There are some important differences here from the system today.
First, fields do not have a mutability associated with them and
neither do unique types.  The idea is that "owned" values are always
typed as `Ty`.  They will inherit the mutability of their owner.

The pointer types, `@ M Ty` and `& M Ty`, each include a mutability
modifier `M` indicating the mutability of the target memory.  There
are three values: the default (immutable), `mut` (mutable), and
`const` (readable).

The `& M Ty` types are intended to model today's reference modes.  I
think you could rewrite the system in terms of modes but I didn't care
to think about it.  Instead, for simplicity, I'll just say that `& M
Ty` types cannot be used within other types but only as the types of
parameters.  An `& M Ty` type can be produced using the `&` operator,
which takes any lvalue.

## Local variables

One big change from today's system is that local variables (though not
parameters) will be given both a mutability and a type.  A local
variable introduced with `let` is immutable and a local variable
introduced with `mut` has mutability `mut`, as per issue #XYZ.

Therefore:

    let x = { f: 3 }; // x has type `imm {f: int}`
    x = { f: 5 };     // error: x has immutable type
    let mut y = { f: 3 }; // y has type `mut {f: int}`
    y = { f: 5 };     // just dandy.
    
In fact, generally speaking all lvalues have an associated mutability
`M`: for local variables, it comes from the manner of their
declaration.  For a deref'd pointer `*v`, it comes from the type of
the pointer `v`.  So if `v` has type `@mut T`, then `*v` has
mutability `mut`, and so on.  For a vector `v[5]`, it comes from the
type of the vector `v`.  For a field `v.f`, it comes from the
mutability of the *record* `v`.  That last part is a bit subtle, and
we'll discuss it below.

You'll note that I didn't give a mutability to unique pointers.  Since
unique pointers are always, well, unique, I don't *think* there's a
point.  Basically you can say they are always mutable.  You can
"borrow" a unique pointer in a function call (i.e., use it as the
value for a parameter of type `&M T`) so long as the parameter has
read-only or mutable type.  In fact we can even allow it to be
borrowed as an immutable type but we have to be a bit clever to be
sure it is not also being borrowed as a mutable type simultaneously.

## Mutability of fields

We can determine the mutability of an lvalue `E.f` like so:

- first, if `E` has pointer type, then the mutability comes from the
  auto-dereferencing.  So if `E` evaluates to a value of type `@mut
  T`, then `E.f` is mutable.
- otherwise, `E` must have record type.  In that case, if `E` is an
  lvalue, we inherit the mutability of `E`.  Finally, if `E` is an
  rvalue, then `f` is mutable (but there isn't much point, is there?
  After all, `E` is some record that is going to be stored to a
  temporary location, never to be referenced again).

