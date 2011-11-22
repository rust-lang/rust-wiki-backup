# Regions in Rust

## Summary

The proposal is incorporate *pointer regions* into the type system.
Pointer regions distinguish pointers based on how long the pointer is
valid.  I personally think a better name is *pointer extents*, but I
have stuck with regions in this proposal as it is the common
terminology.

## Why regions?

This proposal resolves a number of outstanding usability problems
and adds some desired features:

- Address-of operator `&` as in C (aka, local references)
- Returning references
- No implicit copies
- Addressing Dan's bug
- No "modes" on parameters
- Only one generic kind bound: "sendable" (written `send`)

Regions also enable important future use cases:

- References within records
- Typesafe arenas at runtime as an alternative to GC, ref-counting
- Messages containing graphs

Most of these benefits require regions.  Some of them (notably Dan's
bug) are accrued through other changes, such as a shift towards
references as well as some minor restrictions on patterns.

None of these benefits require the alias analysis we have today.  This
is a plus because the precise rules in use by the alias analysis are
hard to define. The alias analysis is still useful and could be
retained in the form of a statement like `freeze(x) {...}`, which
executes a block of code while holding the variable `x` immutable.
Importantly, if the alias analysis proves too coarse to validate the
`freeze` statement, it can simply be removed from the program and then
the program will still compile, albeit with a weaker static
correctness guarantee.

## Pointer types

The set of Rust types under this proposal can be defined formally 
by the nonterminal `t`:

    t = Id                  (nominal types like tags)
      | X                   (type variables)
      | (t*)                (tuples)
      | { (f: t)* }         (record types)
      | int ...             (scalar types)
      | ~r                  (unique pointer)  
      | @r                  (reference counted pointer)
      | R&r                 (pointer with region R)
      
    r = t
      | [t]                 (vector types)
      | str                 (strings)
      | uniq fn(t*) -> t    (unique closure type)
      | fn(t*) -> t         (closure type)
      | block(t*) -> t      (block)

Under this scheme, any type not prefixed by a `~`, `@`, or `R&` is
considered a *value type* (modulo [shorthands](#shorthands)).  The
nonterminal `r` refers to "referencable" types.  Note that some types,
like vectors and strings, have variable size and thus must always be
passed by reference (i.e., a type like `[T]` is illegal, or perhaps
just shorthand for `&[T]`).

## Region parameters

Functions can be parameterized by regions.  Because this is so common
we make use of a very lightweight syntax.  The following function
`get_f()` uses a region parameter `A` for its argument:

    type T = {f: int};
    fn get_f(t: A &T) -> int { ret t.f; }

The region parameter `A` is declared simply by virtue of being used on
a parameter and not already being in scope.  The effect of this
annotation is to allow `get_f()` to operate over any instance of `T`,
regardless of where it is allocated.  

Typically, one would never write a region parameter if it is used only
once.  The following function definition:

    fn get_f(t: &T) -> int { ret t.f; }

is exactly equivalent to the original with the explicit `A` region.
In general, a type like `&T` can be used to specify the most general
possible pointer type.  In the case of parameters, the region defaults
to a fresh region parameter.  Within the body of the function, the
region is inferred based on usage.  

In fact, there are only two cases when one would want to use explicit
region parameters.  The first is to specify that two pointers must be
in the same region, and the second is when returning a reference.
This is particularly useful when a function returns one of its
arguments:

    fn pick_one<A&>(a: A& T, b: A& T) -> A& T {
      if cond { ret a; }
      else { ret b; }
    }

Here, the function `pick_one()` returns either `a` or `b` depending
on some condition.  

## Regions

Rust pointers can fall into the task-local region `@`, an exchange
region, or a stack region.  The task-local region `@` is exactly as it
is today; it is also special because `@` pointers are reference
counted, which introduces an asymmetry in the type system that
requires great caution.

Note that `~` is not a region: instead, it is a way of indicating that
the pointer is located in a unique region only accessible via the
given pointer.

Every function activation yields one or more stack regions.  There is
one for the main part of the function, as well as additional stack
regions for the body of each while loop and for `alt` statements.
Stack regions are used to store the contents of literal records,
vectors, tags and so on.  Each such stack-allocated object will be
allocated at the innermost region at the point in which they are
declared.

> Example:
> 
>     type T = {f:int};
>     fn func() {              // -+ (Region X)
>         let a = {f:1};       //  | (a: X&T)
>         let b = [a];         //  | (b: X&[X&T])
>         while cond {         //  |-+ (Region Y)
>             let c = {f:1};   //  | | (c: Y&T)
>             let d = [c];     //  | | (d: Y&[Y&T])
>             let e = b + d;   //  | | (e: error)
>         }                    //  |-+
>     }                        // -+
> 
> The function `func()` has two regions.  The outermost, `X`, represents
> the function activation as a whole, and the innermost, `Y`, represents
> the while loop body.  The first record (`a`) and vector (`b`) are both
> allocated outside the loop and hence are placed into the region `X`.
> The second set of variables are allocated in the loop in region `Y`.
> When an attempt is made to combine `a` and `c` into one vector, the
> result is a type error.  This is important because it prevents data
> allocated inside the loop from leaking outside, which could cause
> memory errors unless `alloca()` were used. (Note that there is no way
> for code from outside the loop to name the region for the loop body)

Stack regions typically do not have user-defined names but are
assigned names from the compiler based on their line, column.  We may
want to allow a syntax for labeling loops or blocks and thus giving
user-defined names. Such a name would only be in scope within the
block or loop body.

### Capture regions

A unique pointer like `~T` is not in a region `~`; rather, it is in
some distinct region of its own.  To simulate this, any time that a
unique pointer `~T` is assigned to an lvalue of type `R&T` (function
calls and `alt` statements, really), `R` is bound to a fresh region
`X`.

Due to ref-counting, the `@` region is distinct from stack regions and
must be carefully handled.  Therefore, we apply the same capture
treatment to `@` which we apply to `~`.  The intention is to prohibit
a region that appears in multiple places within the function signature
from being bound to a `@` or `~` variable, as this could allow the
callee function to make incorrect assignments in the future
(currently, it would be harmless).

> Example of what would be harmful, assuming types parameterized
> by regions:
>
>     type T<R&> = { f: R&F };
>     fn set_f(t: A&T<A&>, f: A&F) {
>         t.f = f;
>     }
>
> If `set_f()` were invoked with `A` bound to `@`, the ref count would
> not be properly maintained. Bad. Of course, right now this cannot
> happen because we do not allow types parameterized by regions.  But
> this might be a useful feature.

## Inferring region values

Within a function body, a `&T` type will infer the appropriate region.
H-M style unification inference can be used.

## Assigning to region pointers

The contents of a `R& T` pointer can be modified using an assignment:

    *y = new_value;

However, this is only allowed if either:

- `T` is a value type
- or, every field of `T` is mutable.

## Taking the address of something

As in C, the expression `&lvalue` yields a pointer to the location
where the contents of the lvalue are stored.  This pointer is placed
in the stack region of the enclosing allocation.  This is true even if
you take the address of a field in the task-local heap (see below for why).

There are some limitations:

- You cannot take the address of an immutable field.  This could be
  loosened if we decided to include [read-only pointers][javari], but there is
  really no reason to take the address of an immutable field, as its
  contents never change.  If needed, you can read it into a local
  variable and take the address of that.
- You cannot take the address of a field of a unique pointer. You can
  workaround this restriction using an `alt` statement to create a
  temporary alias.

Taking the address of a field of an `@T` object increases the ref
count until the resulting pointer goes out of scope.  This is
typically the end of the containing function, loop iteration, or block
as appropriate.  The pointer is placed into the corresponding stack
region.  This is a bit of a lie, as the data being pointed at does not
technically reside in that stack region, but it is live at least until
the stack region is popped due to the ref count.  This is precisely
the reason that I would prefer a name like *extent*, *duration*, or
*interval* in place of region.

[javari]: http://types.cs.washington.edu/javari/

## Dan's bug

Dan's bug refers to the danger of pattern matching against mutable
data.  Regions alone do not solve Dan's bug; some additional rules are
required.  

First, `alt expr {...}` expects `expr` to have a reference type `R&T`.
This reference will be valid for the duration of the alt:

- If the reference is of type `~T` (or a generic type, which may be
  unique), then the expression is borrowed as described above (hence,
  it must be a local path).  The borrowing rules ensure that the value
  being matched is inaccessible.
- If the reference is of type `@T`, then the reference count is increased.
- If the reference is of type `R&T` where R is a stack region, this stack
  region must enclose the `alt` statement and therefore will not go out
  of scope.  Even if the variables in the expression are reassigned to new
  values, the reference itself is not overwritten.
  
Except for the first case where `expr` is of type `~T`, there is
always the possibility that another alias exists to the same data
which could possibly mutate the contents of the reference.  To prevent
this, the `alt` rules are restricted such that each arm may only match
against *immutable data* unless the expression is of unique type.
This means that mutable fields of records along with mutable arguments
to tags (do such things exist?) cannot be matched; similarly, the
contents of mutable vectors cannot be matched.

Alternatively, we could allow the `copy` keyword in patterns to allow
mutable fields to be matched via a copy:

    type T = { mutable x: X, y: Y };
    fn foo(t: &T) {
        alt t1 {
            { x: copy(x), y: y } {
                // x has type X
                // y has type &Y
            }
        }
    }
    
In fact, making use of sigils *could* remove the trailing "." from
variables.  The idea would be that the pattern "x" is in fact a unary
constructor, not a variable.  A variable must be introduced with
either a `&`, indicating a reference into the matched structure
(requires immutable field or unique type) or a `copy`, indicating a
copy of the value out of the structure.

## Use cases

Here I have written out some of the examples that motivated various
design decisions.  I have used completely explicit annotations and
tried to use the tightest annotations possible, meaning that if a
language features appears it is necessary.

### Take address of local variable

    fn foo() {
       let x = 1;
       let y = &x;
       bar(&x);
       *y = *y + 1;
    }
    fn bar(x: A& int) {
       *x = 3;
    }

### Take address of mutable fields from records on stack, heap, etc

    type T = { f: int };
    fn foo(x: @T) {
       bar(&x.f);

       let y = { f: 1 };
       bar(&y.f);

       // z has type S&int where S is the stack region.
       //
       // Taking this reference increases the ref count on x
       // so that the reference remains valid.
       let z = &x.f;
       *z = *z + 1;

       // w has type S&int too.
       let w = &y.f;

       let i = 0u;
       while i < 10u {
         // u has type L&int where L is the region for
         // this while loop.  The ref count is increased until
         // the current iteration of the while loop finishes
         // (or until u is dead, whichever we prefer).
         let u = &x.f;
       }
    }
    fn bar(f: A& int) {
       *f = *f + 1;
    }

### Prevent stack-allocated structures from leaking out of a loop

    fn foo() {
      let x = [];
      while ... {
         // Error: this structure is allocated in the region for
         // the body of the while loop, whereas the type of x is
         // the region for the entire function.  This is a region
         // mismatch.
         x += { ... };

         // This is ok, though:
         let y = [];
         y += { ... };
         y += { ... };
      }
    }

### Processing unique trees

Using regions, functions, and `alt` statements, it is possible to
create temporary aliases to unique data and process it.  For example, 
imagine a tree type like:

    type tree<T> = {
        left: ~tree;
        right: ~tree;
        data: T;
    };
    
I can now write a recursive processor like this:

    fn map_tree<S,T>(t: tree<S>, f: block(S) => T) -> ~tree<T> {
        // Note: no "~" on type, hence borrow.
        let d = f(t.data);
        let l = map_tree(t.left);
        let r = map_tree(t.right);
        ret ~{left: l, right: r, data: d};
    }

One can also make use of `alt` statements to create block-scoped aliases:

    fn proc_tree<T>(t: ~tree<T>) {
        alt (t.left, t.right) {
            (l, r) { 
                // Here, l and r have type {L,R}& tree<t>, where L and R are
                // fresh regions.  The original "t" pointer is inaccessible.
            }
        }
    }

### Mutable borrowing

To support borrowing in the sense of punching a temporary hole in the
data structure, one can use a library like:

    type borrowable<T> = { mutable item: ~option::t<~T> };
    fn borrow<uniq S,T>(b: &borrowable<S,T>, f: block<&T,S>) -> S {
      let prior = (b.item <-> ~option::none);
      let res = alt prior {
        // the path prior is considered borrowed here:
        option::none    { fail "Already borrowed!"; }
        option::some(t) { f(t) } // t has type R&T for some fresh variable R
      }
      b.item <-> prior;
      ret res;
    }
    
## Limitations

Here are some examples of things we can't do because the system is too
limited.  Generally, the problem is that parameterized types would be
required (nothing in this proposal prevents parameterized types from
being added).

### Place parameters into stack-local objects (requires parameterized types)

        type ctx<X&, Y&> = { x: X&X, mutable y: Y&Y };
        fn foo(x: A&X, y: B&Y) {
            let ctx<A&, B&> = { x: x, y: y, ... };
            bar(ctx);
        }
        fn bar(ctx: A&ctx<B&, C&>) {
           // What can bar NOT do:
           //
           // bar cannot modify fields of ctx.x or ctx.y whose types
           // involve & the precise region of ctx is not known.
           //
           // bar cannot modify the field y beacuse the precise region of
           // ctx is not known.
        }

   Here parameterized types are used to store pointers to `x` and `y` in a
   record on the stack.  The annotation burden is high, which motivates
   better defaults and notation.  Making full use of defaults and inference
   would allow the following:

        type ctx<X&, Y&> = { x: X&X, mutable y: Y&Y };
        fn foo(x: X, y: Y) {
            let ctx = { x: x, y: y, ... };
            bar(ctx);
        }
        fn bar(ctx<&, &>: ctx) {...}

## Expansions or possible modifications

### Explicit regions

I think it would be nice to be able to allocate memory pools and then
create data in them.  This would fit very nicely with the region
system described here.

Arenas provide for a middle-ground between reference counting and
garbage collection.  For example, it would be possible to develop both
traditional "free at once" areans as well as garbage-collected arenas.

Finally, arenas provide new capabilities, such as the ability to send
graphs of data between tasks rather than merely trees.  The idea is
that the arena itself would be tracked using a unique pointer, but the
data within need not be.  When the arena is sent, the typestate system
would ensure that no existing uses of data in the arena can be used.
The existing unique pointer infrastructure can be used to guarantee
that no live aliases to the arena nor the data contained in the arena
exist. One note of caution is that sending graphs in a distributed
settings---or even between processes---is somewhat painful.

Runtime arenas would also require further investigation as to the
smoothest (and most ergonomic) way to integrate them into the type
system.

### Region parameters in types

Type declarations can also be parameterized by regions.  The syntax is
the same as the syntax for functions.  The following type declaration,
for example, declares a tree that can be placed into multiple regions:

    type tree<D, R&> = {
        left: R& option<R& T<R&>>;
        right: R& option<R& T<R&>>;
        data: D;
    };
    
Using a declaration like the one above, a function could construct a small
tree that is allocated entirely in its stack frame:

    fn construct_tree() {
        let one = {
            left: option::none,
            right: option::none,
            data: 22
        };
        let root = {
            left: option::some(one),
            right: option::none,
            data: 44;
        }
    }
    
Similarly, one could declare a function that processes a tree, regardless
of where it is allocated:

    fn sum_tree<R&>(t: tree<int, R&>) -> int {
        fn opt<R&>(t: R& option<R& tree<int, R&>>) -> int {
            ret alt t {
                option::none    { 0 }
                option::some(t) { opt(t.left) + opt(t.right) + t.data }
            };
        }
        ret opt(option::some(t));
    }
    
#### Defaults and shorthands for unspecified region parameters

It is generally not necessary to specify region parameters.  The compiler
will use defaults and inference to fill in the gap, depending on the context:

- Function parameters are annotated with a fresh region by default.
- Missing region arguments from a type use the region of the type
  itself, if possible.
  - If the type appears as an argument to another type, then the region is taken
    from the enclosing type.
  - It is an error to omit region arguments for a unique type (`~T`).
- Local variables use inference.

These defaults mean that most of the annotations present in the prior examples
could be removed.  For example, the function `get_int()` could be written:

    type T = {f: int};
    fn get_f(t: &T) -> int { ret t.f; }

Similarly, the function `sum_tree()` can be written like so:
    
    fn sum_tree(t: &tree<int>) -> int {
        fn opt(t: &option<&tree<int>>) -> int {
            ret alt t {
                option::none    { 0 }
                option::some(t) { opt(t.left) + opt(t.right) + t.data }
            };
        }
        ret opt(option::some(t));
    }
    
The defaults for type parameters are useful but (naturally) not always what
you want.  In particular, a common need is to have fresh regions as the value
for parameters as well.  For this purpose, a bare `&` can be used within a
function declaration to create fresh regions:

    fn sum_tree_2(t: tree<int, &>) -> int { ... }

The function `sum_tree_2()` is equivalent to this fully qualified version:
    
    fn sum_tree_2<A&,B&>(t: A& tree<int, B&>) -> int { ... }

### Wildcard pointers in records

The current proposal requires that all records specify the precise
region for all fields, even if that region is a vairable.  Earlier
versions included 'wildcard pointers', written `&T`.  A wildcard
pointer indicates a pointer in some region that outlives the region of
the record itself, but the precise region itself is not known.  

These types are both more and less powerful than the parameterized
types in this proposal.  They allow for more concise notation but also
allow for types that can be both unique and region'd.  However, they
lose information and can generate errors when dereferencing long
chains of fields.  The mechanics of 

    type ctx = { f: &T; ... };
    fn clam17(f: &T) {
      let ctx = { f: f, ... };
      do_stuff(ctx);
    }
    fn do_stuff(ctx: &ctx) { ... }

The meaning of a &T in a record depends on the region containing
record itself.  Given a field x.f declared as &T, when used as an
rvalue:

- If `x` has type `@S`, then `x.f` has type `@T`
- If `x` has type `~S`, then `x.f` has type `~T`
- Otherwise, `x` has type `R&T` where `R` is a fresh region variable

Assigning to `&T` fields is only legal when the precise region of the record
itself is known.  Therefore, a field `x.f` can be assigned:

- If `x` has type `@S`, then `x.f` can be assigned an `@T` value
- If `x` has type `~S`, then `x.f` can be assigned a `~T` value
- If `x` has type `R&S`, where `R` is a stack frame region for the current
  function, then `x.f` can be assigned from
  - an `@T` value
  - a `R&T` value
  - a `Q&T` value where `Q` is some region variable

On the other hand, `x.f` cannot be assigned if:
- `x` has type `R&S` where `R` is a region parameter of the function.  A region
  parameter might refer to the heap, for example.


