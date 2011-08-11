(These are Tim's sketchy notes attempting to capture discussions with Patrick and Dave about constrained types, as well as other thoughts.)

## Problem

Mutation of local variables that have constrained types explicitly assigned to them means that programmers can introduce all kinds of type assertions that can't be checked in the typechecker (because it doesn't do typestate), and can't be checked in typestate (because these assertions are non-local).

For example, consider the following program:

    type odd = int : odd(*);
    fn odd(int x) -> bool { ... }
    ...

    check odd(y);
    let x : odd = y;
    y = 4;

This program is bad. But why is it bad? [TO BE CONTINUED]

## Solution

Disallow constrained type names as an explicit type on the LHS of a declaration. Constrained type names can only appear as data structure field types, and function argument types.

Introduce nominal records. Constraints can only be on nominal records, not on tuples or structural records. 

## Unresolved questions

What to do about variants? Can individual tags have constraints, as well as product types? (What about constraints that say "this data structure was built with tag X"? That could be faked with predicates, but maybe syntax for it would be nice.)

## Other notes

Typestate _has_ to happen after typechecking -- unless type inference has already occurred, we don't have access to information about which nodes have constrained types, and thus can't check the relevant constraints.

## To Do (for Tim)

I'm not even sure whether constrained types that aren't record types (e.g. `odd` above) actually work right now. So checking whether they work would be the first thing for me to do.