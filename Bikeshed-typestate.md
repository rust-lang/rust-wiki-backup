Tim's notes after discussing the current state of typestate with Dave:

* The notion of "immutable" type becomes problematic in the presence of abstract types and interfaces. How do you know whether a type is immutable?

* Immutable types are no longer as central a part of Rust as they use to be.

Possible solutions:

* "Read-only" view of a data structure

* Region types (a la Cyclone)

* Refinement types: Limit to a few common idioms that are known to the compiler. For example, subset of the variants of an ADT (datasort refinements). Maybe simple arithmetic (e.g. for bounds checking)

* Analogous problem to gradual typing where you push downcasts (that is, constraint checks) to all call sites; bad for usability. Expressing richer invariants in APIs suggests pushing lots of checks to call sites. It's an open question how many checks can be eliminated through clever typechecking and data structure design.

Dealing with mutable data:

* Subtyping (one caller's view of a data structure need not be as constrained as another's, as long as no illicit writes that break someone else's view are happening)

* Wrappers (check that the invariant still holds whenever an update occurs); possibly bad for performance

* A wrapper is kind of like an upcast: "turn this into something that upholds the invariant"