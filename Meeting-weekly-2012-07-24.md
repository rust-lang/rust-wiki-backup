Rust meeting

- LLVM bump (graydon)
- hash function (graydon)
- buildbot status (graydon)
- remove modes? (graydon)
- mutability / sendable maps (nmatsakis)
- minor nomenclature shift (graydon)
- operator overloading w/ coherence (pcwalton)
- linked failure status, interface (bblum)
- runtime-enabled condition variables (bblum)

## LLVM Bump
- graydon: various weird LLVM bugs, one of which prevents building w/ gcc 4.7...
- graydon: ...and hence all modern linuxes and also mingw cannot build
- graydon: will migrate to a more modern LLVM
- eholk: can we use a canned LLVM release?
- graydon: going to try 3.1 or 3.2
- pcwalton: but we will still have patches on top of that, due to GC

## Hash fn
- graydon: everyone writes them and all of them use bad hash fns
- graydon: turns out that there is a nice, new generic hash fn with nice properties that we can hopefully use everywhere (short input, cryptographic, can be keyed to a pseudo-random # per hashtable)
- graydon: so if you find yourself writing a hash fn, stop, and try this one instead
- graydon: basically the same construct used in the SHA3 candidates, real fast
- nmatsakis: what about uninitialized / unused data
- graydon: current just hashes a bucket of bytes, code adapting our data will have that problem
- graydon: can we maybe just not have uninitialized bits?
- nmatsakis: well, padding etc --- not unachievable just painful, runtime cost
- pcwalton: a minor complication, one optimization I would like to start doing at some point is to save space when allocation a @variant that is immutably bound.  Important for DOM nodes since size varies considerably.
- nmatsakis: yeah, that'll complicate something like is_pod
- graydon: deal with that when we come to it
- pcwalton: how complete is the visitor? would love if we could throw away shape glue
- graydon: haven't tested profoundly, but one thing it definitely doesn't do: interface instances  ...
- graydon: a little bit of a trade between code that drives visitation and the client code, since client code can decide when to recurse
- graydon: not sure if it handles interfaces properly right now
- pcwalton: one of the first things we could do is use it for hashing
- pcwalton: the way I envision this working eventually is to have a hash trait with a default impl based on visitors
- pcwalton: like Haskell's deriving but more extensible
- nmatsakis: boxed interfaces are self-describing... should be able to work like other @T
- graydon: what about closures?
- nmatsakis: fn@ and fn~ should work, fn& no
- pcwalton: maybe we don't even need that?  
- nmatsakis: by definition, everything in there is a borrowed ptr so GC won't care...
- graydon: should basically work
- nmatsakis: what about comparing pairs of values?
- graydon: basic interface just visits types and knows nothing else
- graydon: built on top of that various examples in the test suite, one of which visits pairs of pointers
- graydon: now that the snapshot contains reflection interface should be able to move those into libcore
- graydon: idea was to have a library in core called reflect which would abstract over the usual cases (walk data, value, etc)

## Buildbot status

- graydon: got a security review, security team did not find it particularly secure (sadly)
- graydon: so in limbo right now discussing how to mitigate that problem, most likely by moving things (bot master?) into the data center
- graydon: current plan is perhaps to publicly expose only HTTP

## Removing modes

- Need a transition plan
- ...
- nmatsakis: Summary of the plan
    - make an attribute that changes the default
    - send out an e-mail explaining the semantics of the options
    - also, develop a plan for things like vec::each() and vec::each_ptr()

## Mutabilty / sending maps

- ...
- nmatsakis: will write up a blog post

## Nomenclature Shift

- graydon: rename shared boxes to managed boxes
- graydon: could then rename ARC to shared

## Operator Overloading and Coherence

- pcwalton: if we add ability to export metadata... (which we need)
- pcwalton: what we could do is say that there are some "magic" traits that implement the built-in operators
- pcwalton: for example, an add trait Add<T> with a method plus(T) and perhaps a plusEquals(T) method
- pcwalton: if you want to override an operator, you implement that trait for your type
- pcwalton: to reduce magic-ness, these would be in core and annotated with some special "intrinsic-plus-trait" annotation.  only one such trait is permitted across all crates.
- graydon: what does this buy us?
- pcwalton: this buys us a way to do operator-overloading in a coherent setting
- pcwalton: ensures that you can never have multiple methods in scope and never need to import the plus trait
- graydon: this is a corrolary to inherent methods on a type...?
- pcwalton: well, resolve is already somewhat involved, when you see a method call `a.b()`, resolve goes around and finds the traits that define `b()` (considering also inherent methods) and comes up with a candidate set.  Type check is then based on this candidate set.
- pcwalton: so this would change it so that if you have `a + b` the `+` method would always yield the method from the plus trait
- pcwalton: to me the benefits are
    - never have to import traits for plus
    - plus-equals as a default method
    - simpler grammer (no methods)
    - mildly simpler resolve
- eholk: some of the benefits of plusEquals() could be obtained by plus 
- pcwalton: ...might be that uniques are moved by default...
- eholk: ...or at least uniques by default...
- eholk: I sort of prefer writing `fn+()` 
- pcwalton: we can keep that around too, whatever people prefer
- eholk: would be nice to be able to key on the type of the RHS
- pcwalton: that's real, real overloading
- graydon: I don't want to do that unless we have to
- nmatsakis: we *kind of* have it, in the sense that you can setup ifaces that lead to overloading, but we fail to solve them so we don't have it

## Failure

- bblum: one thing that doesn't work is multiple generations of unidirectional failure
- bblum: e.g. if grandkid dies but kid doesn't parent should die (But doesn't now)
- bblum: You can:
    - link tasks bidirectionally (create a task group)
    - spawn tasks unlinked (their own group)
    - spawn tasks supervised (new task group, but if parent task group dies, new group dies)
        (spawning with failure prop error from self to new task)
- bblum: method chaining interface is in (e.g., `task().unlinked()....spawn()`)
- bblum: various top-level spawns exist as shorthand (e.g., `spawn_unlinked()`)
- bblum: migrated code to use that instead of old builder interface, currently migrating servo
- nmatsakis: what happens if I request notification, the task completes, and then someone else in the task group dies
- bblum: non-deterministic, though you should only get one message
- nmatsakis: maybe we can add an enhancement so that you can get a message when a task-group dies instead of an individual message? then you can choose what you want
- bblum: task group is not a reified thing
- bblum: graydon wanted ability to spawn into a task group, but I couldn't quite get it to make sense
- ...
- bblum: on a related note, exclusive arcs are currently protected by this `rust_cond_lock` thing which is basically a pthread mutex
- bblum: condition variable behavior is totally broken (and currently disable) but mutex behavior is only a little bit broken.  It does disable mutual exclusion but cannot accommodate yields since scheduler is not aware of locks that you hold.
- pcwalton: one more reason that exclusive arcs are evil
- bblum: same problem for cond. vars, since this really requires scheduler interfacing
- bblum: I have a scheme for fixing this but code will have to be very, very clever to interact "properly" with task killing
- nmatsakis: what is desired interaction with task killing?
- bblum: let's say you've taken a cond lock and waited, and then your task group fails:
    - you should awaken and them immediately die
- bblum: if you are contending on a mutex
- ... complicated semantics discussion ensues ...
- take it offline!
