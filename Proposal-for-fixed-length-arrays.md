It is generally acknowledged that unique vectors and strings are not
working out well in practice, due to implicit copies and limitations
of unique types.  This proposal advances the idea of fixed length
arrays in place of growable vectors.  The resulting system is quite
similar to O'Caml or Java: bounds-checked arrays are the basic
building block and growable vectors are moved into the library.

## Array creation

Arrays can be created either via literals `[v1, v2, v3]` or via the
use of creation functions.  Literals result in an array of the given
size which is stored on the stack.  There are two creation functions,
each of which creates an array of dynamic length:

    fn create_val<copy T>(size: uint, v: T) -> new [T];
    fn create<copy T>(size: uint, b: block(uint) -> new T) -> new [T];

The function `create_val()` creates an array of a given size where
each value of the array is initially equal to `v`.  The second creates
an array where the initial value of the array at index `i` is given by
`b(i)`.

## Mutability

As with vectors now, arrays can either be immutable (the default),
mutable, or read-only (`const`).  An immutable array cannot be
modified by anyone.  A read-only array is a reference to an array thay
may be mutated by someone else.

Functions returning `new` arrays by value may be used to create either
mutable or immutable arrays at the callers' choice.  For example:

    let f: [mutable int] = arr::create_val(256, 0);
    
Creates a mutable `int` array of length `256`, where the initial value
is `0`.  We could also allow unique arrays to become mutable and
immutable if we wished.

## References into arrays

The syntax `&arr[5]` can be used to take a reference into an array.  This
works the same as other references.

## Growable vectors

Growable vectors can be implemented as a Rust library.  There are some
debates as to how deeply they should be built into the language itself.
A proposal built entirely in Rust would be something like:

    mod vec {
        type t<T> = {
            mutable data: ~[option::t<T>];
            mutable size: uint;
            mutable capacity: uint;
        };
        fn create<T>() -> t<T>;
    }
    
This has the disadvantage of requiring option types in the array.
Perhaps a better choice would be to use an unsafe, opaque
implementation.

## Runtime implications

Returning a array by value---as done by the `create_val()` and
`create()` functions---is a bit tricky.  Typically, when returning an
aggregate by value, the caller allocates the space and the callee
fills it in.  This allows the caller to decide whether the returned
value shall be placed into a ref-counted box, on the exchange heap, in
the stack, etc.  However, because the size of an array is not known
until runtime, this system does not work.  Therefore, we generalize
the ABI so that instead of passing in a pointer, the caller passes in
either a pointer or a heap data structure, using a tagged pointer to
distinguish.  The callee can then either allocate the required space
or use the pointer.  An alternative would be to pass in two words
(i.e., a Rust closure).

Unfortunately, for callees with statically sized return types, the
callee cannot statically know whether it will be required to allocate
space or not; the caller might be invoking the callee generically, in
which case the caller cannot know whether the return type has dynamic
size.  Potentially this could be addressed by wrappers and the like
but it is unclear if it will be a bottleneck.  Monomorphization would
address the problem.

As an alternative, we might make functions return via pointer by
default (as today) and use the new keyword only for those cases the
size of the return value is not statically known.
