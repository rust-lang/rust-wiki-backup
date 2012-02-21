## Attending

Jesse, Brian, Niko, Dave, Tim, Patrick, Graydon, Marijn

## Project management

* Graydon: posted yesterday RFC process document and some notes on retagging in issue tracker, until I realized you can't do AND queries
* Graydon: GitHub says this is known and being worked on
* Graydon: I added some clustering tags for the time being
* Graydon: worst case we could go back to bugzilla
* Jesse: GitHub issues app on Heroku has querying support. (searching for "a-attributes i-papercut" on http://githubissues.heroku.com/#mozilla/rust does the intersection correctly)
* Marijn: even if not, there's an API; we could write a search engine ourselves
* Graydon: only thing missing is negation, but this will help
* Dave: seems inevitable they'll eventually fix GitHub issues
* Graydon: yeah, they fixed a ton of things last time, and they have tons of enterprise customers, so this is all on their list
* Niko: question: didn't quite understand what the wiki said -- if we settle on nominating for closing an issue, what tag do we use?
* Graydon: oops, forgot that; I'll make it a B tag
* Graydon: oh, I did create one: B-IsThisDone
* Niko: sounds like "Did we do this already"
* Graydon: yeah, I'll do B-ShouldClose too

## Status updates

* Niko: Tim, what's the status on classes?
* Tim: some implementation, front-end stuff, no methods yet; still very early on
* Niko: definitely wanting traits
* Dave: don't you need classes before traits?
* Niko: independent: can be used on impls
* Patrick: traits want CCI to be really useful
* Niko: that's true, gated on me
* Niko: my status: serialization mostly done, hoping to start deserialization today
* Niko: don't know if everyone knows: I've been blogging about region options, and I was going to make a table about different options
* Graydon: that would be super useful, and your blog posts have been extremely tantalizing
* Niko: I've been writing the posts because I couldn't make the table till I understand the different options
* Niko: I feel like there's a continuum and the current system is at one end
* Graydon: yeah, I think we can nudge it a bit further w/o overwhelming the user
* Graydon: one thing I want to point out: you talk about available-everywhere regions, I wanted to point out the constant region would be a super-useful one
* Niko: good point
* Graydon: other status updates? I'm working one export globbing, library and OS work
* Marijn: optimization work mostly complete; haven't finished vtable part; made much more complicated by having non-monomorphized; think I'll leave that till that's gone; may do bug-hunting in the meanwhile
* Graydon: always a good idea
* Niko: hopefully we can bring those things together next week
* Brian: rewrote task API over weekend; not dramatically different from before; updates to make room for wanted things
* Brian: working on getting rustdoc finished; also working on parallelizing it (not strictly necessary, but fun)
* Graydon: noticed an AST server growing in rustdoc; nice place to prototype it; if it works can put it in the compiler
* Jesse: actually faster with parallelization?
* Brian: not yet; need to use borrowable pointers for the par_map; currently copying structures
* Jesse: can we do borrowing without null?
* Niko: it's hard
* Graydon: could change to unique, hand it out as unsafe pointers, when they give it back return ownership
* Niko: I think that's something we'll want to talk about; hard part is you have to do it in some lib that knows the tasks are done
* Jesse: should par_map be part of std lib?
* Brian: we'll see; obviously we'll eventually want lots of concurrency primitives

## Refinements

* Marijn: one thing I want to discuss: can people agree to just removing preconditions from std lib functions? I suspect Tim is the only one who'll have a problem with this
* Marijn: not adding anything, making lib hard to use; until typestate matures we should remove them
* Graydon: I tend to agree
* Patrick: me too; it would be good to tackle in a more holistic way at some point; I do like the idea of dynamically checked contracts (with names); something worth thinking about
* Dave: trade-off I see: dynamic approach leads to lack of clarity about what can fail and what can cost at runtime; lint mode simply puts this decision on the programmer
* Tim: I'm ok with removing preconditions until typestate gets fixed, but I'm concerned that we won't fix typestate
* Dave: my only concern is that we may not find a solution
* Tim: sure, as long as it's an explicit decision, rather than just getting ignored
* Graydon: yeah. e.g., if it's gonna be a DbC system, we should make it that (e.g., remove purity). I'm not quite ready to abandon the static approach. I do really want that
* Dave: maybe restrict to subcases that we can reason about more robustly: non-empty vectors, subcases of enums
* Niko: I like static reasoning, definitely in favor of seeing what we can express
* Patrick: this sounds more like types than refinements
* Tim: yeah, this is closer to the datasort refinements I proposed earlier
* Patrick: to me, important question is not whether refinements are useful but ... well, another question anyway is whether the control flow will be useful
* Dave: I think it could still play a role if there's still useful information that can be conveyed to the type system via `if`
* Graydon: kind of depends how expression-y you want the language to be
* Dave: I think it still comes up even in an expression-y language
* Graydon: informal poll of preferences: do you want to bother with static reasoning e.g. datasort refinements?
* (most raise hands, not Marijn)
* Graydon: another informal poll: do you think we'll need to abandon the user-specified predicates being used for static refinements?
* (Patrick, Niko, Dave raise hands)
* Patrick: we definitely want some particular patterns of static refinement of info, but I don't see the user-specified refinement types working well
* Graydon: I talked with the Hermes people a while back; they used it for message-passing, so I'd hoped it would be useful for us too; but if it's not working out for us I'm ok with focusing on the stuff where we need it
* Graydon: I still think the control-flow part could play a role
* Patrick: it's still useful for init checking
* Dave: even if it's just used for that, I still think it pays for itself
* Graydon: so we'll go ahead and remove the refinements from the stdlib
* Jesse: change to assertions at top?
* Graydon: yeah; we can talk down the road about enhancing typestate further or turning it into DbC
* Graydon: we're not at the point of a full-on assault of that subsystem yet

## Unsafe blocks

* Jesse: I have some concerns about unsafe blocks
* Brian: Jesse & I were discussing: lots of things we do in unsafe blocks that inhibit things the compiler is supposed to be doing but isn't
* Brian: specifically, won't be able to yield nor cycle-collect; hadn't considered this till recently, but seems pretty clear that's the case
* Jesse: also unwinding if you get a failure
* Brian: right, can't unwind
* Niko: unsafe::leak which leaves refcount unadjusted
* Niko: not entirely clear to me why that should prevent cycle collection etc
* Brian: b/c yielding can fail; if we unwind, we would drop the ref twice
* Niko: yielding can fail b/c it might check an error state
* Brian: linked failure; child fails
* Niko: so leak has to be done atomically, which we could do if we're more careful with move mode
* Graydon: mm-hmm
* Niko: my question is, is it only refcounting, or are there other problems?
* Brian: don't know. specifically familiar with this case
* Patrick: C# has this thing called pinning: in order to interact w/ pointers in unsafe way, have to pin the pointer; tells the GC not to touch it; could stuff like that be helpful?
* Niko: maybe
* Brian: possibly. wouldn't solve problem of not being able to fail
* Niko: kinda feel like if it's only refcounting, that's addressible in another way; if there are other problems, it'd be good to know
* Jesse: is it reasonable to say: if you fail in unsafe block and compiler doesn't know how to unwind, you'll lose the whole process?
* Brian: so much important code written in unsafe blocks... and you could get a kill signal at any time
* Graydon: you're saying any time there's an unsafe block on the stack atack at all?
* Brian: yeah
* Niko: you've only given one example...
* Patrick: can we use resources? rewrite unsafe blocks to be failure-safe somehow?
* Niko: resources and failure have a checkered relationship; if there's a failure in a resource, etc etc
* Graydon: I had a document written up a long time ago about this
* Niko: all I'm saying is, I've tried using resources this way and had leaks
* Graydon: yeah, there's a bunch of interlocking things here, and it's extremely tricky and super hard to debug
* Graydon: one example: comm system has to have clear lifecycles for everything
* Graydon: needs to be a version of failure that works in all contexts on all platforms
* Graydon: we should have better answers than "the process shuts down" -- that's us saying we didn't do our homework
* Niko: similar case is when you have a failure and there's C code on the stack; in that case maybe we really can throw up our hands
* Dave: is it not fair to say that unsafe code is just a more convenient way to write C code?
* Niko: I'm not yet convinced this is an issue; leaking refcounts is tricky business, and if you're mucking with pointers, it's tricky
* Patrick: there are multiple things here: reason we abort in C calls is that we don't know that that C code has any sort of failure-recovery mechanism at all; if there were a way to show that C code were all exception-safe, or returned an error code, we could do something different; but a lot of times that's not true of C code; but that's not true of Rust unsafe code: we actually have hope of giving it sane failure semantics; maybe some way to tag unsafe code as not failure-safe; don't think it's the case that unsafe code must abort, because it does have a sane failure semantics
* Niko: that's exactly what I think
* Graydon: let me rephrase succinctly: in Rust side, we can assume programmers are aware of failure; on C side, we can't; conclusion: unsafe Rust is opt-out, and C code is opt-in
* Jesse: in the C code or in the Rust binding for the C code?
* Graydon: could go either way
* Niko: there's a wide range of cases to handle
* Graydon: I think it's reasonable, for example, to have C have a dynamic way to interface with failure system
* Niko: JSAPI, for example, uses return codes
* Graydon: two sides to that: how does C code inform Rust callers that it's failed; how does Rust code that's been called back from C inform C that it's failed
* Niko: right
* Graydon: two possibilities: mark function saying this C code is aware of failures and has this mechanism; declare this C code is failure-irrelevant, no big deal to just toss it out
* Niko: what I wanted to ask: could imagine marking C code as failure-safe, could be Rust code's responsibility to take return code, or could be built into the runtime
* Niko: just want to see if we can orthogonalize two concerns
* Graydon: ability to set "this task is failing" and check "is this task failing" from C side should be sufficient, so long as when you unwind through a C frame you clean up; gets into implementation of the unwind system
* Graydon: keep in mind, portable unwind system has a dynamic component

## Structured exception handling

* Graydon: if we stop using DWARF failure system, we could have a uniform failure model; downside is that exceptions are no longer zero-cost
* Graydon: what are people's gut feelings on this? do you like SEH?
* Graydon: three options
* a. only use SEH on Windows, otherwise use some C API
* b. switch to using that API on all platforms for uniformity
* c. give up on allowing C code to clean up after itself
* Patrick: I want to add: Brian & I have talked about this, SEH cost is not as high as you might think; cost is only applicable for resources; for refcounts, unilaterally blow away on failure, and for unique pointer you can also unilaterally blow away; so only have to run destructors
* Niko: would have to track which unique ptrs owned by process
* Patrick: yeah, some bookkeeping, but not that much; so it seems SEH is tolerable when it's only for resource destructors
* Graydon: so SEH everywhere, uniform interface everywhere
* Patrick: yeah
* Niko: I agree
* Niko: Graydon: are you saying the C code would have to call these things, so you'd have to write C code wrappers for libraries that don't know about Rust?
* Graydon: yes, you'd have to write a C wrapper that adapts; usually just one function; not that hard to tell users how to do it
* Niko: seems like it'd be enough to set an annotation on C callback, something like canfail, on exit from function check the RustCurrentTaskIsFailing and continue to propagate failure; is that true?
* Graydon: would be assuming all C code can fail; always checking that flag on return
* Niko: might want to try to optimize that
* Graydon: flag check doesn't sound that bad to me
* Niko: no, true. main thing is we want to be able to catch the error, let the C code propagate it for a little while, and then keep going
* Graydon: yeah
* Niko: as long as that's what we're talking about, I'm fine. I'd rather you don't have to write C code every time
* Graydon: anyone not wanna do SEH everywhere approach?
* (crickets)
* Patrick: binary size was also an issue
* Graydon: yeah, it mattered on cell phones. but then everything matters on cell phones
* Graydon: so sounds like concensus to try it out and see
* Niko: yeah
* Jesse: so this is how Rust itself handles failures, in addition to C code
* Graydon: yeah; basically giving up on DWARF zero-cost exceptions
* Jesse: alternative on windows?
* Graydon: windows has something special, metadata blobs that look something like DWARF; has something that all Windows debuggers use
* Jesse: it's zero-cost in the not-throwing case?
* Graydon: yeah, never truly zero-cost, but it's about not throwing
* Patrick: and to reiterate, if we're clever, this should only cost for resource destructors, which are rare
* Niko: yeah, in C++ it's mostly for memory management
* Jesse: well, in Firefox we use auto classes (RAII) for security invariants as well
* Graydon: Patrick: when you say clever, how clever?
* Patrick: well, failure is non-recoverable, and all RC data is task-local, so you can indiscriminantly blow away all task-local data once destructors have run
* Graydon: you do have to find destructors in heap
* Niko: well, we can walk the heap
* Graydon: you also have to drop unique ptrs
* Patrick: right, need bookkeeping to see which uniques belong to the task
* Niko: walk the stack to find roots
* Graydon: one way or another, yeah
* Graydon: what's happening in our landing pads now?
* Brian: basically they run cleanup code that our other code runs anyway
* Graydon: so this is a pretty substantial change to codegen; all landing pads will be eliminated
* Dave: won't this change the order of destructors?
* Niko: doesn't bother me
* Patrick: you can compute which RC's were from the stack just based on their counts
* Niko: how important is it to preserve that?
* Graydon: that I don't think is terribly important; concept is the only ordering they provide is the order inside them; but stack destructors should happen inside-out
* Dave: other question: keeping stuff alive longer b/c they get dropped *after* destructors run?
* Graydon: yeah, it's a corner case; originally had two failure modes, soft failure and hard failure; if a destuctor enters hard failure, no more destructors run during that destructor, go back into soft failure after completing that destructor
* Jesse: ISTM we shouldn't allow resources inside destructors
* Niko: we can't do that; could call a function
* Graydon: analogous to Java; exception in finally overrides
* Graydon: same concept, yeah; not currently implemented
* Jesse: not-running-some-destructors sounds like serious leaks, or getting security checks wrong
* Graydon: we'll need a design document, but I believe there's a solution
* Graydon: Brian, are you okay with experiment with SEH everywhere?
* Brian: yep, I'm fine with that
* Graydon: I don't think users are stressing the failure system yet; more of a long-term thing we need a plan for for developers
* Brian: the Windows situation is a little embarrassing right now
* Graydon: SEH also makes it way easier to debug; breakpoints in SEH is doable, with DWARF it's impossible in my experience
* Brian: I've done it
* Niko: Brian's made of stronger stuff than the rest of us
* Jesse: is compiled Rust code fast enough that we could do meaningful comparisons of DWARF vs SEH?
* Patrick: yes, but you have to be careful what kind of program you're using
* Graydon: don't think we'll be able to do an exhaustive comparison, but a gut feeling comparison would be important, like is rustc perf still acceptible; ideally it should get smaller
* Niko: I would even expect it to get slightly faster
* Brian: we took a big hit for DWARF unwinding; I'd expect we could do better by rolling our own
* Patrick: well, be wary of comparisons; may make compiler faster but not other programs
* Graydon: it's a tough decision to make strictly on perf; the perf space is enormous; should base it more on what we want to achieve; perf should just be aiming at "reasonable"
* Brian: Servo will care about perf a lot
* Niko: particularly messaging; latency of messages directly affects the architecture
* Jesse: do we have a perf test suite?
* Brian: yes, but we never check them; I'd like to set up something better before we make a huge push on performance
* Niko: data is good
