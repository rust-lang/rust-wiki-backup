(These are Tim's sketchy notes attempting to capture discussions with Patrick and Dave about constrained types, as well as other thoughts.)

## Problem

How should the typechecker generate constraints for typestate to solve, specifically those that arise from uses of constrained types?

Mutation of local variables that have constrained types explicitly assigned to them means that programmers can introduce all kinds of type assertions that can't be checked in the typechecker (because it doesn't do typestate), and can't be checked in typestate (because these assertions are non-local).

For example, consider the following program:

    type odd = int : odd(*);
    fn odd(int x) -> bool { ... }
    ...

    check odd(y);
    let x : odd = y;

This program is perfectly acceptable. On the other hand, consider this program (with the same definitions):

    let x : odd = y;

where we omit the `check`. The typechecker or typestate pass (one or the other) needs to be able to reject this program. 

## Design space

There are three possible solutions:

1. Allow declaring `x` as type `odd`, but with the caveat that the semantics of `let x : odd = y` don't imply that `odd(x)` holds everywhere in the scope of `x`.
2. Allow declaring `x` as type `odd`, and statically require every write `x = y` (for any `y`) to be preceded (on every incoming path) by `check odd(y);`
3. Disallow `let` declarations with explicit type `odd` (or any other constrained type).

(2) is troublesome as no other part of the language says anything about constraints on mutable data. It would be a bit inconsistent to handle constrained types differently from predicate constraints. Also, this solution requires a pretty wide interface between the typechecker and typestate: there has to be a way for the typechecker to record hints saying "there needs to be a check for this constraint holding for this subexpression because it's an RHS of an assignment to something with a constrained type", since the need for the check isn't locally syntactically apparent.

(1) is confusing because in this scenario, the static type of a declaration for `x` -- `odd` in this example -- doesn't make it immediately obvious what type `x` has in *every* context. If `x` is declared `odd`, but on some control paths it actually has type `int`, that's an unusual use of a static type declaration.

## Solution

So, Solution (3) seems like the most appealing one: Disallow constrained type names as an explicit type on the LHS of a declaration. Constrained type names can only appear as data structure field types, and function argument types. 

Introduce nominal records. Constraints can only be on nominal records, not on tuples or structural records. (This makes constrained types analogous to calls to functions that have constraints: we add a single introduction form for constrained types, otherwise known as a data constructor.)

## Unresolved questions

* Disallowing mutation on constrained variables seems to suggest that, for consistency, function arguments that are declared with constrained types should also be made immutable. But that introduces another question: in general, argument slots are mutable. Is making an exception for arguments with constrained types too confusing?

* Then, if arguments with constrained types are immutable, for consistency, arguments mentioned in a function precondition constraint should also be immutable. That is, in the following two example function signatures (where the definitions of the `odd` type and the `odd` predicate are as above):

        fn f(x:odd) { ... }
        fn g(x:int) : odd(x) { ... }

it would be strange if `f`'s argument had to be immutable and `g`'s was mutable, or vice versa.

* Each of the three possibilities in the "design space" section also applies for arguments that have constrained types. For consistency, it would be desirable to apply a consistent policy to arguments and to locals.

* Solution (3) poses a refactoring hazard: adding a perfectly good constraint on `x` to an existing function `f` with argument `x` could render the code ill-typed if `f`'s body assigns to `x`.

*  For functions, the constraint applies to the _caller's_ view of the argument, but mutation applies to the _callee's_ view of the argument.

* What to do about variants? Can individual tags have constraints, as well as product types? (What about constraints that say "this data structure was built with tag X"? That could be faked with predicates, but maybe syntax for it would be nice.)

* Nominal records: could we avoid adding them to the language, and instead, encode them as product types using the ```tag``` construct and a field that has a structural record type?

## Another possibility

This possibility requires a more pervasive change to the language, but what if argument slots were immutable and Rust had an additional assignment operator, `:=`? The semantics of `a := b` -- where `b` must be an argument of the nearest lexically enclosing function, and `a` must be a local variable that's not an argument -- are that the compiler must statically reject this code if it's impossible to do copy propagation, eliminating the binding for `a` while preserving all constraints. That is, if `b` has constraints and the function body ever mutates `a` again, this code would be rejected, as a constraint could be violated that way. Call it a "virtual assignment" operator.

## Other notes

Typestate _has_ to happen after typechecking -- unless type inference has already occurred, we don't have access to information about which nodes have constrained types, and thus can't check the relevant constraints.

Open question: should the compiler be able to use refinements (for example, "this refinement on tag T never has constructor a") to eliminate some `alt` checks? This introduces some more machinery and would mean, for the first time, that type soundness involves arguments about the meaning of user-defined predicates.

## To Do (for Tim)

I'm not even sure whether constrained types that aren't record types (e.g. `odd` above) actually work right now. So checking whether they work would be the first thing for me to do.