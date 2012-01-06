## Attending

Brian, Dave, Marijn, Niko, Patrick

## Last use analysis

* Dave: concerned it makes significant move-vs-copy decision based on non-trivial flow analysis
* Marijn: important optimization, and should be done anyway, but user-visible since it accepts otherwise rejected program
* Dave: Niko and Patrick have been discussing an idea that makes copyability a structural decision rather than using flow analysis
* Marijn: don't want to have to explicitly copy all over the place
* Niko: wouldn't explicit copy scalars, but generics conservatively requiring explicit copies could become a pain point
* Marijn: about 1/3 of the stdlib generics do copies; could potentially annotate type params as copy to alleviate constant annotations. We could investigate a more explicit model for some types, but this is a very basic analysis and seems just plain helpful
* Dave: ok, we can investigate the combination; I'm a bit concerned about allowing many types to be auto-copy, because whenever you distinguish kinds you complicate generics

(After meeting, Niko and Dave discussed what the last-use analysis was doing and understood better that it's orthogonal to the alternative copy semantics Patrick and Niko have been discussing.)

## Outparams

* Patrick: another concern is cost of using outparams everywhere
* Marijn: LLVM optimizes this well; doesn't seem like a real perf problem

## Unique vectors and strings

* Marijn: another issue is how to go forward with vectors and strings
* Patrick: Niko and I have been thinking about this lately too
* Marijn: please post your ideas
* Niko: executive summary: change built-in vectors to be:
 a. fixed-length arrays (Ocaml) or
 a. double-indirection (Python)
* Niko: leaning towards a) for default, b) in lib
* Marijn: can we make this as efficient? e.g., for realloc?
* Niko: programmer can explicitly contain with unique pointers
* Marijn: what about refs inside growable vecs?
* Niko: disallowed
* Patrick: that depends; if you have refcounted buffers and can't shrink, it can be made to work
* Niko: that's a sort of unpredictable semantics
* Patrick: with mutation, it gets hairy
* Marijn: can check not shrinkable when unique
* Patrick: could have runtime hand you a buffer rather than a ref inside the buffer
* Niko: dynamic-size arrays need to be task-local or unique
* Marijn: at any rate, could become less powerful and efficient than current system if you can't take refs in as many places
