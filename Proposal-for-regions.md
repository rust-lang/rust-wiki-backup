# Regions in Rust

## Note

Rust has regions, but not exactly as they are described here.

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
- No "modes" on parameters (though `move` could still have a role to
  play)
- No need for approximate alias analysis
- Better support for stack allocation

Regions also enable important future use cases:

- Typesafe memory pools:
  - faster performance as with Marijn's recent experiments
  - messages containing graphs

## Pointer types

The set of Rust types under this proposal can be defined formally 
by the nonterminal `t`:

    t = Eid                 (enums)
      | X                   (type variables)
      | (t*)                (tuples)
      | { (f: t)* }         (record types)
      | int ...             (scalar types)
      | ~r                  (unique pointer)  
      | @r                  (box pointer)
      | R&r                 (pointer with region R)
      | fn b (t*) -> t      (closure of various kinds)
      | Iid b               (interfaces)
      
    b = ~                   (unique bound)
      | @                   (box bound)
      | R&                  (specific region bound)
      |                     (any bound)
      
    r = t
      | [t]                 (vector types)
      | str                 (strings)

The nonterminal `r` represents types that can be addressed by pointer.
Note that that certain types, such as arrays and strings, have
variable size and so can only be referenced by pointer.

## Region parameters

Functions can be parameterized by regions.  Because this is so common
we make use of a very lightweight syntax.  The following function
`get_f()` uses a region parameter `A` for its argument:

    type T = {f: int};
    fn get_f(t: A& T) -> int { ret t.f; }

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
region parameters.  The first is to specify that two parameters must
be in the same region, and the second is when returning a reference.

Here are some examples where explicit region annotations are required:

> Example 1: **Selecting from one of two arguments**
>
>      fn pick_one(a: A& T, b: A& T) -> A& T {
>          if cond { ret a; }
>          else { ret b; }
>      }

> Example 2: **Returning a reference into a structure**
>
>      type T = { f: int };
>      fn select_field(a: A& T) -> A& int {
>          ret &a.f;
>      }
>
> Note: supporting examples like these definitely complicates
> the memory management story.  I am inclined not to support them,
> in fact, and instead to code up such an example as:
>
>      fn with_field(a: A& T, f: fn(&int)) {
>          f(&a.t);
>      }
>
> Otherwise, the type checker must ensure that `a` remains live
> as long as the returned value is live.

## Regions

### Stack regions

Every function activation yields one or more stack regions.  There is
one for the main part of the function, as well as additional stack
regions for each block within the loop as well as for alt statements.
Stack regions are used to store the contents of literal records,
vectors, tags and so on.  Each such stack-allocated object will be
allocated at the innermost region at the point in which they are
declared.

Stack regions have a subregion relationship.  There is a subtyping
relationship induced between pointer types based on their region
parameters.  So, imagine a function whose stack region is `X&` and
which contains a block with region `Y&`.  Then a pointer `X&T` is a
subtype of `Y&T`.  The intution is that any time when a `Y&T` pointer
is valid, the `X&T` pointer must also be valid.

> Example:
> 
>     type T = {f:int};
>     fn func() {              // -+ (Region X)
>         let a = &{f:1};      //  | (a: X&T)
>         let b = &[a];        //  | (b: X&[X&T])
>         while cond {         //  |-+ (Region Y)
>             let c = &{f:1};  //  | | (c: Y&T)
>             let d = &[c];    //  | | (d: Y&[Y&T])
>             let e = b + d;   //  | | (e: ~[Y&T]
>             b = b + d;       //  | | (error)
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

### Heap regions

Rust pointers can fall into the task-local region `@` or an exchange
region.  The task-local region `@` is exactly as it is today; it is
also special because `@` pointers are reference counted, which
introduces an asymmetry in the type system that requires great
caution.

Note that `~` is not a region: instead, it is a way of indicating that
the pointer is located in a unique region only accessible via the
given pointer.

### Capture regions and unique pointers

A unique pointer like `~T` is not in a region `~`; rather, it is in
some distinct region of its own.  To simulate this, any time that a
unique pointer `~T` is assigned to an lvalue of type `R&T` (function
calls and `alt` statements, really), `R` is bound to a fresh region
`X`.

### "Reparenting:" Taking the address of something

As in C, the expression `&lvalue` yields a pointer of type `R&T` to
the location where the contents of the lvalue are stored.  The region
`R` with which the resulting pointer is associated is always the
innermost enclosing stack region.

It might seem surprising that the resulting pointer is always in a
stack region.  After all, imagine you have a box `x` of type `@T` and
you do `&x.f`: you might expect that the type of this expression would
be in the `@` region.  The reason that it is not is that the region
actually reflects the extent for which the pointer is valid.  Taking
the address of a field of a box will cause the box's ref count to be
increased until the end of the current block, thus ensuring that the
block remains valid.  But after the current block, the block might be
freed, and so the validity of the pointer cannot be guaranteed.

Another interesting case is taking the address of a field or local
variable of unique type.  We wish to prevent the possibility that the
compiler believes the variable is unique but in fact there exists an
indirect alias. The simplest thing is simply to prohibit taking the
address of values of unique type. XXX not entirely true, &x.f where x
has unique type is also bad.  Need a more clever rule. In general this
is ok, but we must ensure that if the container from which the field
was read has unique type, it is considered borrowed.  For more
details, see the section on uniqueness.

## Inferring region values

Within a function body, a `&T` type will infer the appropriate region.
Our current inference system should mostly be adequate, though
accommodating subregions might require a bit of effort.  Moving to an
inference system that is more aware of subtyping would help.

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

    fn map_tree<S,T>(t: &tree<S>, f: fn(S) => T) -> ~tree<T> {
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
    
## Expansions or possible modifications

### Self regions

A simple variant of parameterized types would be to allow `self`
regions.  `self` refers to the same region as the enclosing record.
It can be used to define "re-usable" types that can be allocated in
multiple locations:

    enum tree<T>  = _tree<T>;
    type _tree<T> = {
        data: T,
        left: option<self& tree<T>>,
        right: option<self& tree<T>>
    }

Here a `tree<T>` instance could be safely placed in the `@` region, in
a unique region, or on the stack:

    fn foo() {
        let x = &{ data: 1, left: none, right: none };
        let y = &{ data: 2, left: none, right: none };
        let z = &{ data: 3, left: some(x), right: some(y) };
        assert sum(z) == 6;
    }

    fn bar() {
        let x = @{ data: 1, left: none, right: none };
        let y = @{ data: 2, left: none, right: none };
        let z = @{ data: 3, left: some(x), right: some(y) };
        assert sum(z) == 6;
    }

    fn sum(t: &tree<int>) -> int {
        let l = option::default(option::map(t.left, sum), 0);
        let r = option::default(option::map(t.right, sum), 0);
        l + r + t.data
    }
    
Note that the routine `sum()` can be used both for data on the stack
and data in the heap.
    
It can also allow for examples where a context object is created on
the stack.  The context object should point to a bunch of other
values.  It should be possible to freely pass it around by pointer
amongst various functions.

    type ctxt = {s: self& S,
                 t: self& T,
                 u: self& U };
    fn foo(s: &S, t: &T, u: &U) {
        let cx = &{ s: s, t: t, u: u };
        do_lots_of_processing(cx)
    }
    
    fn do_lots_of_processing(cx: &ctxt) {
        ...
        do_lots_more_processing(cx);
        ...
    }

    fn do_lots_more_processing(cx: &ctxt) {
        ...
    }

This could apply in numerous places in our compiler.  I am not *sure*
whether it could work in trans, due to our pervasive use of boxes, but
I suspect it could be made to work if we restructured how block
contexts are setup.

There is however a limitation required to preserve soundness: We can
only safely allow `self&` regions in immutable fields (and only if
immutable really means immutable).  Otherwise, reparenting and
ref-counting can both cause problems, as shown in these examples:

    type S = { mutable v: self& int };
    type T = { mutable s: self& S };
    fn foo(x: @T) {
        let s = &x.s; // type: S& 
        let f = &x.f; // type: XXX is this ok? is it only ref-counting?
        let f = &x.left; // type: S&mut option<S& tree<int>>
    }

### Safe memory pools

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
    
We have to be careful, though, with soundness if we include
subregions.  Region parameters can only appear in immutable fields or
else variance will cause issues.

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
