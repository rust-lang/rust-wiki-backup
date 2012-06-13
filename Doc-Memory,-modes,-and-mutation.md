**A guide for the perplexed.**

_Do you find yourself writing random punctuation marks in front of types in hopes of making the errors go away? I sure do. So I decided to write some documentation by randomly hitting keys on the keyboard, and having other people fix the errors._

# Concepts
## Slots
Slots are places where a value can live in memory, like the `x` in `let x = 4` or `{y: 3, x: 2}`.

## References
References are values that "point to" slots. Like all values, they live in (presumably different) slots.

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

# `@`
The expression `@2` produces a reference to a immutable slot in the heap. Executing that expression will result in an allocation, but no visible side-effects. Its type is `@int`. 

If you want mutability (which is pretty much the only reason for putting integers into the heap), `@mut 2` has the type `@mut int`. If you have a variable `x` with that value and type, you can change it with `*x = 3`.
# `~`
Locally, `~2` is the same as `@2`, but it also means that no other references to that slot exist. So, the following function:
````Ruby
fn f(x: ~int) -> {
  *x = 8;
  do_lots_of_complicated_stuff_with_callbacks_and_mutations_aplenty();
  ret *x
}
````
...always returns `8`, because no one else has a reference to that slot to change it.

To enforce this, the compiler will give you errors any time you attempt to copy a reference with `~` type.
# `copy`

# `[]`