## Attending
Brian, Dave, Marijn, Niko, Patrick

## Objects

Discussed Patrick's draft design of an OO system based on zero-inheritance non-virtual classes (type + implementation), structural interfaces (type but not implementation), and traits (implementation but not type).

* Patrick: everyone seems to agree it makes sense to make interfaces nominal; I didn't see how to do `I1 + I2` but we can maybe do `interface I3 = I1 + I2`
* Dave: if I is recursive, not sure what it means to add it to another interface, i.e., what the recursive references mean; might be solvable
* Patrick: or maybe just punt; not sure it's a necessary feature
* Patrick: stages of implementation, where we have something useful at each stage:
 1. classes
 1. interfaces
 1. traits
* Niko: now what about unifying classes and records?
* Patrick: yes, as Marijn says, !EIAO is a bad design smell
* Marijn: unifying classes and records doesn't resolve the issue of other primitive types; you're still in Java-land
* Patrick: solves some but not all; I'm not opposed to the idea of Scala implicits
* Dave: my only concern is modularity issues with overloading
* Niko: Scala implicits make coercions an importable thing; scoped imports restrict coercions to lexical scopes
* Marijn: does it do operator overloading?
* Niko: yes: they use extensively in collections library
* Niko: also, implicit parameters; e.g., `sort` takes an implicit `<` function
* Marijn: this sounds good; not necessary for first stage, but if everyone agrees it's a good goal, then the proposed object system sounds promising to me
* Niko: implicits help with context passing too; A takes implicit, can implicitly pass to B too
* Patrick: I'll send a description of the system to rust-dev

## Regions

* Patrick: background: subsumes some cases of alias analysis; precedent is from Cyclone
* Patrick: basic idea: first-class reference types, type system enforces that references cannot escape their liveness
* Marijn: can you send papers?
* Patrick: yes
* Dave: might help to write an overview, too, since the lit may go off on tangents
* Patrick: yeah, the original lit tried to replace GC entirely, which ended up making it unnecessarily complex; we're only interested in safe references
* Marijn: I'm open to ideas; just had to fix some of stdlib b/c of a hole in the alias analysis
* Patrick: I also have ideas for doing safe pools and arenas

## Kinds

Question about default kind bounds on polymorphic type parameters.

* Marijn: pinned-by-default is most general, but you often want shared
 * infer by default?
 * warn if over-constrained?
* Patrick: what about shared by default?
* Marijn: lthat's what I did, but may over-constrain
* Patrick: I'm nervous about inferring based on the body; we've avoided doing that anywhere else
* Niko: almost always want shared
* Marijn: except, not for datatypes
* Patrick: shorten to `pin` and `uniq`?
* Marijn: blech! but meh
* Dave: you can't turn off the warnings
* Marijn: so maybe we make shared default but warn if over-constrained; you turn off the warning if you explicitly say shared
