## Attending
Brian, Dave, Marijn, Niko, Patrick

## Unsafe blocks
* Niko:
 * can tag functions and blocks as unsafe
 * unsafe functions only usable from other unsafe functions and blocks
 * can't use unsafe functions as first-class values
 * status:
  * missing restriction to second-class
  * need to audit all operations, see which need to be restricted
  * currently all natives tagged as unsafe
* Marijn: maybe not the right default; forces us to write wrappers
* Niko: probably don't want to have to add a `safe` keyword
* Patrick: maybe come up with heuristics based on whether they take pointers?
* Brian: calling into LLVM could cause an assert to happen just about anywhere
* Marijn: can't always prove things are safe, but don't want to have to infect everything
* _resolved_ - safe by default for native

## Typestate

* Marijn:
 * not working out in some places where we lose info due to mutation
 * can we stop using it where it isn't appropriate?
* Niko: should look at Scala's approach to subtyping for cases like subsets of disjoint unions
* Dave: this is like ad-hoc unions
* Patrick: should visit when we revisit object system after v0.1
* Marijn: do we have subtyping?
* Patrick: kind of, only a very little bit due to `mutable?` (which we probably want to change)
* _resolved_ - where they are proliferating, just change to assertions if we're hitting them and eliminate if we aren't

## Miscellany

* Marijn:
 * tutorial in Markdown? - fine with everyone
 * destructuring assignment? - fine with everyone, but need to think about syntax (preferably no sigil/keyword)
 * mix assignment and binding in a single term?
  * Dave: I don't like mixing binding and reference/assignment in one form
  * Marijn: but this saves you from destructuring twice
  * Patrick: well, you can bind the names w/o initializing up front, and then just use assignment
  * Marijn: that's uglier
  * Dave: subjective; my instinct says it's not
  * Marijn: I'll work on it and keep thinking about it
