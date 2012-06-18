**A guide for the perplexed.**

_Do you find yourself writing random punctuation marks in front of types in hopes of making the errors go away? I sure do. So I decided to write some documentation by randomly hitting keys on the keyboard, and having other people fix the errors._

# Concepts
## Slots
Slots are places where a value can live in memory, like the `x` in `let x = 4` or `{y: 3, x: 2}`.

## References
References are values that "point to" slots. As they are also values, they live in (presumably different) slots.

There is no way to ask for the "address of" a slot, in order to generate a reference to it. If you want a reference, you need to generate it in the heap with `@` or `~`, ensuring that it lives long enough to prevent dangling references. (This doesn't apply to the reference mode of parameters, `let`s, etc., described below. Because they have known, limited lifespans, the compiler can tell when it is safe for them to reference slots on the stack or inside structures.)

## Mutability
If a slot is mutable, its contents may change:
````Ruby
let x_imm = 4;
let mut x_mut = 4;
x_mut = 456;
// x_imm = 456; // illegal
````

Mutability is shallow, not deep: a mutable slot may contain a reference to a structure with immutable slots, and an immutable slot may contain a reference to a structure with mutable slots.

## Initialization
If a slot is uninitialized, you may not read it. For example, if you have `let x: int;`, you may not read `x` until you assign to it. This property is enforced at compile-time, so it is conservative.

The name is perhaps a little misleading; in Rust, it is possible for a slot to become uninitialized if it loses the "right" to access its contents, via moving.

## Modes
Rust currently supports parameter-passing modes. There are five or six of them: `+`, `-`, `++`, `&`, `&&`, and one that you get if you don't write any funny symbols. What do they mean? It is a mystery.

## Regions
A region is a space in memory with a shared lifetime. That is, everything gets deallocated together.

# Data/types
## Boxes: `@`
The expression `@2` produces a reference to a immutable slot in the heap. Executing that expression will result in an allocation, but no visible side-effects. Its type is `@int`. 

If you want mutability (which is pretty much the only reason for putting integers into the heap), `@mut 2` has the type `@mut int`. If you have a variable `x` with that value and type, you can change it with `*x = 3`.
## Uniquely-referenced boxes: `~`
Locally, `~2` is the same as `@2`, but it also means that no other references to that slot exist. So, the following function:
````Ruby
fn f(x: ~int) -> int {
  *x = 8;
  do_lots_of_complicated_stuff_with_callbacks_and_mutations_aplenty();
  ret *x
}
````
...always returns `8`, because no one else has a reference to that slot to change it.

To enforce this, the compiler will give you errors if you attempt to implicitly copy a value with `~` as a type. Explicit copies will also recursively copy through `~`s, to avoid duplication.

## `[]`

# Manipulating things
## Binding: `let`
The statement `let x: int = 3` introduces an immutable slot, named `x`, and initializes it with `3`. You can use arbitrary patterns on the left of a `let`, and it will destructure the given value.

## Assignment: `=`
The left-hand side of an `=` must designate a mutable slot. The right-hand side indicates what to put in that slot. 

## Transferring things around by reference: `let`, `alt`, and function invocations

## Copying: `copy`
If you are handling a piece of data that might potentially be passed by reference, and you want to avoid that (either because `~` forbids it, or because you want to restrict the visibility of mutations), you can write `copy`. Copy is "as shallow as possible", which means that it descends through `~`s, but stops at `@`, merely copying the reference.

## Moving: `<-`
_I'll figure out what is going on here later:_
````
<pauls> It's not possible to move out only one field of a record, right?
 Is there any workaround other than recreating the whole record?
<eholk> pauls: if it's the last use, possibly
<eholk> you can also swap a mutable option with none
<nmatsakis> pauls: actually
<nmatsakis> it is possible
 in some cases
 e.g., if you have let x = {f: ~3};
 you can then write let {f: f} <- x;
 but this only works if the record itself is movable, of course.
````