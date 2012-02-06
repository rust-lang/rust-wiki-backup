## Attending

Niko, Dave, Graydon, Tim, Marijn, Brian

## Exhaustiveness

* Tim: making everything exhaustive will add a lot of default cases; should we do it now
* Niko: I want the check; would rather have it now and decide how to clean it up later
* Graydon: I concur; prefer to have the logic in place
* Tim: I'll take ownership of adapting it in the future

## Priorities

* Niko: I plan to integrate the iteration library; just some helpers; apply `map`/`filter` to anything that takes a last argument
* Niko: I'd like to talk about an RFC for `break`/`continue` in blocks
* Graydon: I want to clean up stdlib soon; people will depend on that sooner than they'll depend on semantic corner cases
* Niko: that's why I want to push iter now
* Brian: will this replace the other stuff?
* Niko: could replace most but not `foldr`; I guess you could do `foldl` with reverse iter
* Graydon: I'm developing in Win7 now, so I can work on Windows issues with I/O and the OS module
* Niko: do you have a design in mind?
* Graydon: well, it's misdesigned now; organized with old assumptions about the crate system
* Graydon: I'm curious -- there's been some work on libuv bindings; what's that?
* Brian: Patrick and Donovan have bindings for SpiderMonkey hooked into Rust and DOM.js; I did the bare minimum libuv bindings needed to get XHR working
* Brian: the bindings are sloppy; named uvtmp so it's obvious I don't like them; but OK to punt on uv since Servo is now unblocked

## Scheduling

* Graydon: you talked about first-class schedulers, i.e., bringing back domains; I've been thinking about this
* Brian: yep
* Niko: I agree
* Brian: there's a lot of bindings that want to be able to block
* Graydon: current scheduler: thread mapping is incorrect; tons of people are running into this
* Niko: SpiderMonkey is a prominent one that we'll have to deal with that has particular threading requirements; I was looking at a lib for actors in Java (Kilim) that got a lot of mileage out of having this control
* Niko: we should address C calling into Rust for 0.2 -- Patrick has a proposal
* Niko: he looked into Go, and their solution wouldn't work for us
* Niko: Patrick's proposal:
 * first export a C API for sending messages to Rust
 * should use use that for the complex cases
 * simple cases: if Rust fails, abort process
 * small callbacks work
 * do we want to allow Rust to block?
 * while blocked, own that C code
* Graydon: that's exactly my concerns:
 1. how to express closures to C
 1. unwinding
 1. scheduling
* Niko: there's aren't really many possibilities
* Graydon: yeah, just block the current thread
* Niko: if we take that approach, scheduler should be able to steal tasks
* Marijn: that only works on multicore, though?
* Graydon: no, multiple OS threads
* Marijn: oh right
* Niko: simple way to implement is thread pool
* Graydon: that's the easiest way (what I did initially)
* Niko: and most portable
* Niko: how hard would it be to say if you block we'll spin up a new thread in the pool and give it a scheduler? I'd like to redesign the scheduler
* Graydon: yeah, those 2 things should happen simultaneously
* Niko: we have to be able to dynamically grow and move tasks
* Graydon: just keep 1 extra thread around, add one whenever you run out
* Niko: yeah, there are known techniques for this
* Graydon: we don't need to flood the cores; in a sense lightweight tasking is a bit old-fashioned; a lot of OSes can have lots of threads these days
* Niko: I did this for my dissertation; had 3k idle threads and decreasing that didn't seem to affect perf
* Graydon: not sure this is possible by March
* Niko: how about sub-tasks by March? i.e., a sub-task for C in its own thread
* Graydon: a little unsure about sending messages
* Niko: that's why it's in a separate thread
* Graydon: this needs a written-up proposal
* Niko: I think it's important
* Graydon: agree it's important, just not sure if it's 0.2 or 0.3

## Priorities for 0.2

* Graydon: I have OS bindings that don't destroy PATH variable; should I call this 0.1.1?
* Dave: what stuff is 0.2?
* Niko: regions; more design work to be done
* Graydon: I asked a question on the list about regions (are they first-class?) and didn't get a response
* Niko: they're close to first-class, but more expressive
* Graydon: just keep in mind the cautionary tale of Cyclone
* Brian: Patrick wants monomorphism
* Niko: yes, we should start work on that
* Graydon: I'm find working on it, but don't hold up 0.2; my hands are full
* Marijn: first step is cross-crate inlining
* Niko: Patrick started a little on it; generalized the compiler to be able to do more than one crate at a time
* Niko: maybe want to encode AST in a more structured way for compatibility
* Graydon: what happens if you do that?
* Niko: replace the normal crate reader
* Graydon: don't want to recompile the whole crate each time!
* Niko: forest of AST's; don't have separate paths for in-crate vs out-of-crate
* Graydon: just make sure to do it lazily!
* Niko: yes; ideally parse on a function-by-function basis
* Graydon: items are mostly distinct; but within import context (the current imports) -- perhaps this is a tolerable mess
* Graydon: could build index structure for representation; initially pretty-print the source, call that representation 0, then could move to other representations like gzip or something else later
* Niko: that's what we've come to; 1.0 vs 2.0 version issues
* Graydon: I would like to at least bake the language version in compiled crates
* Graydon: also, there's a bug on file for expressing warning levels as attributes
* Tim: for individual functions?
* Graydon: modules, functions, crates, whatever; better than compiler flags
* Brian: some bugs related to that
* Niko: could also possibly be used to flag non-exhaustive matches
* Graydon: well, within reason
* Marijn: if short, not-too-ugly attribute name could be a good way to decorate alts
* Niko: my big things for 0.2:
 * monomorphization
 * regions
 * library work (OS, iter)
 * calling Rust from C
 * Cargo?
* Brian: perf issues easy to get done
* Niko: been working on box alloc issue
* Marijn: I could take that off your hands
* Niko: talking about big hash map of alloc'ed boxes; moving to linked list; I can just finish it

## Classes

* Marijn: new classes proposal fits with interfaces
* Graydon: other than nominal records, is there anything to it?
* Niko: start with nominal records, then move on to methods
* Tim: I'll implement
* Niko: also would like traits
* Marijn: kind of like default methods in Haskell?
* Niko: not sure, we can discuss offline
* Marijn: integrate impls more; I'll just do something and everyone can evaluate it; can import * impls, not necessarily imported at top-level by default, but we can try both ways
