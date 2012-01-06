I propose a unique pointer system similar to
[Haller's work for Scala][haller]. Any value with a unique type `~T`
is always statically guaranteed to be either dead (i.e., will never be
used again), inactive (i.e., cannot currently be used), or the only
pointer to the given data or to the interior of the given data. Unique
types are intended to be used for sending messages but do not have to
be general enough to be used everywhere within the system.

The typestate system tracks the state of each *local path* at each
point in the program.  A local path is a path `a.b...z` where `a` is a
local variable and each prefix `a`, `a.b`, ..., `a...y` has either
unique or value type.  Each local path can either be *available*,
*consumed*, *borrowed*, or *accessed*.  The meaning of each category
will be explained shortly.

[haller]: http://lamp.epfl.ch/~phaller/capabilities.html

## Moves, copies, and swaps

Unique pointers cannot be implicitly copied, which means that they
cannot be directly assigned to other locations.  There is a unary
operator `move` which can be used to "release" a local path so that it
can be assigned elsewhere.  After a move the local path and all its
prefixes are no longer accessible.  The `ret` statement in a function
that returns a unique type also performs an implicit `move`.

> An example showing moves of local paths:
>
>     type X = { f: ~F };
>     fn foo1(x: ~X) {
>         bar(x);         // Illegal, implicit move.
>         bar(move x);   // Legal
>         bar(move x);   // Illegal, already moved.
>     }
>     fn foo2(x: ~X) {
>         bar(move x.f); // Legal
>         bar(move x);   // Illegal, x.f already moved
>     }
>     fn bar(x: ~X) {...}

It is also possible to swap two locations using the `<->` operator.
Swaps can be used to extract unique values from mutable fields of
unique type.

> An example showing swaps:
> 
>     type X = { f: ~F };
>     fn foo3(x: ~X, f: ~F) -> ~F {
>         f <-> x.f;      // Extract x.f value
>         ret f;          // Return the value (implicit move)
>     }

Finally, the `copy` operator can be applied to unique types.  The
result is a deep copy, as elsewhere.

## Borrowing

While unique values cannot be assigned as other values are, they can
be *borrowed* as part of a method call or `alt` statement.  Borrowing
creates temporary aliases of the unique pointer for the duration of
the statement.  Only a local path `a...z` may be borrowed.  While it
is borrowed, the path `a...z` is inaccessible.  Furthermore, all
prefixes of the path (`a`, `a.b`, ..., `a...y`) are considered to be
*accessed*.  Accessed paths may only be used as the prefixes of other
borrowed paths.  Borrowing is a more general form of the `let!`
constructor from [Walder's original work][walder] on linear types.

[walder]: http://homepages.inf.ed.ac.uk/wadler/topics/linear-logic.html

In fact, for `alt` statements the rules of borrowing and accessing are
somewhat more subtle.  The expr being matched against is considered
borrowed when it is stored into a variable and accessed when it is
deconstructed using a pattern.  When the variable is deconstructed
against a record pattern, the matched values inside the pattern are
considered borrowed.

> An example showing borrowing in function calls and alt statements:
>
>    fn foo() {
>      let x: ~X = { y: ~{...}, z: ~{...} };
>
>      // During the next statement, x is accessed, x.y and x.z are borrowed
>      bar(x.y, x.z);
>
>      bar(x, x);   // Illegal: x cannot be borrowed twice
>      bar(x, x.y); // Illegal: x cannot be both borrowed and accessed
>
>      // During the alt, x is accessed and x.y is borrowed:
>      alt x.y {
>        y { 
>          alt x { ... } // Illegal, x is already accessed
>          alt x.z { ... } // Legal
>          bar(y, x.z);    // Legal
>          bar(x.y, x.z);  // Illegal: x.y is already borrowed
>          bar(x, x.z);    // Illegal: x is accessed, cannot be borrowed
>        }
>      }
>
>      alt x {
>        { y: y, z: z } { // x is accessed, x.y and x.z are borrowed
>          ...
>        }
>      }
>    }
>
>    fn bar(y: &Y, z: &Z) { ... }

*Possible change:* we might be able to allow values to be borrowed
twice, or borrowed and accessed.  After all, current rules guarantee
that a borrowed value is unique, which is more than we need.
