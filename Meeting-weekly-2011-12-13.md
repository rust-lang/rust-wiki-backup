## Attending

Brian, Graydon, Dave, Niko, Patrick

## No implicit copies

* Graydon: haven't looked at it enough yet
* Dave: we'll talk more when you get a chance

## Unique closures

* Niko: dilemma: wanted `unique <: shared <: blocks` but unsafe (blocks have RC env, could upcast unique and copy)
* Niko: looking into making a global tydesc cache; should hopefully be able to minimize locking
* Brian: generic bare functions will still need a boxed env
* Niko: I was thinking we'd copy
* Niko: I found a lot of modes cause crashes (exercising shape code and cycle collector); could spend time fixing these
* Patrick: don't spend forever on these bugs
* Brian: CC might be collecting when things are half-constructed
* Patrick: allocation was supposed to zero out pointers during construction
* Niko: otherwise, need safe points; would prefer we don't have to zero (expensive)
* Graydon: try `calloc` instead of `malloc` and see if it fixes; would be a sign that's the problem
* Patrick: maybe treat funcalls as safe points? may not be safe if you're constructing an object via funcalls and DPS
* Niko: might want maps indicating what's not yet initialized
* Patrick: something in the runtime; long-term could go in the stack map
* Niko: medium term just use some function calls
* Patrick: short term just zero
* Niko: meantime, Brian is right that global tydescs aren't enough; need to handle sending bare generics
* Graydon: I'm nervous about this; global caches smell like misdesign
* Dave: is this only short-term?
* Niko: unique closures _want_ to be on the exchange heap
* Graydon: why aren't we doing that now?
* Niko: I want to avoid the proliferation of function types
* Dave: wait, why unique at bottom? why not top?
* Niko: blocks are more restricted
* Patrick: unique closures need to be one-shot
* Graydon: why?
* Patrick: because they drop/move their arguments
* Graydon: that doesn't seem so bad
* Niko: but it's not what you want
* Graydon: one-shot sounds much harder
* Brian: I don't like the one-shot idea
* Graydon: but we don't have deinitializing
* Niko: move is deinitializing
* Graydon: right, right. but swapping is an option
* Niko: we could possibly build wrappers that take data and make it communicable
* Graydon: I just feel we're designing without clear use cases here
* Dave: well, but if there's a soundness problem because unique closures can't subtype the other kinds of functions, we have to fix it
* Graydon: what is the soundness problem?
* Niko: well, maybe you can with blocks
* Patrick: the problem comes from binding move-mode arguments; we could just say you can't do that
* Dave: what about move-mode locals, though?
* Niko: you already can't move upvars
* Niko: but still, you can't allow uniques to be subtypes of shared functions
* Graydon: that was always my intention to disallow that
* Brian: it's still a different representation of environment
* Niko: blocks gin up a dummy RC; could do the same for unique; or when casting lambda to block, just move the pointer past the RC; have to be a little careful about alias analysis; essentially, a block is just a ref to a closure
* Brian: so then how do we do generic functions with bound tydescs as uniques?
* Niko: we can't... maybe `bind` could yield one... might want distinguished syntax
* Graydon: have to differentiate constructors for different types
* Niko: I'm inclined to say `bind` yields shared
* Graydon: take a step back... now that we have lambdas, `bind` feels vestigial
* Dave: may still be ergonomic
* Patrick: since we don't have ML-style auto-currying, it's nice to have a currying operator
* Graydon: ok, well, just don't get hung up on syntactic details
* Brian: if bare fns capturing tydescs can't be treated as uniques...
* Niko: that's ok, just have to copy them into exchange heap
* Niko: I'm inclined not to allow bare fns... only, lame...
* Brian: still want not to create a lambda for every spawn
* Niko: no reason that can't work, I just need to dig into it

## Type classes

* Graydon: OK for Marijn to start laying groundwork?
* Patrick: definitely in favor of replacing kinds conceptually with type classes; and would like some sort of "category" or static type classes; but I'm less sure about dynamic type classes for a few reasons:
 * original Haskell type classes have problem of defeating separate compilation
 * multiple implementations have the "hashtable problem"
 * type-level solutions, but solvable in OO style by just passing a closure
* Dave: I haven't given up hope for the dynamic case
* Patrick: not sure it's worth solving since you can take closures
* Dave: I agree, but I think the dynamic can still work; we just still have open issues so it's too soon to commit
* Graydon: just want the basic static infrastructure
(all agree)

## Libraries

* Graydon: fair warning: there will be churn; new snapshot coming, libs will be versioned; not sure yet how it will integrate with OS lib versioning
