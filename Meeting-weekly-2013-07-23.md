## Agenda

- OSCON tomorrow
- MPS    
- metrics, benchmarks, ratchets and reporting
- small code compile time
- "unsafe" on extern fns
- "Self" in impls
- "Drop" trait self parameter
- *mut
- mutable captures
- &const
- for ... in ...
- library function naming (get/get_ref/unwrap/map/map_consume/iter/...)

## OSCON Tomorrow

- G: In case you don't know, it's tomorrow. Mozilla has a booth and several contributors are there.
- G: Will be quiet Wed/Thu/Fri this week as many of us will be missing.
- G: Congratulations to everyone who pulled it together

## MPS

- G: Memory pool system
- G: Old and well-developed pluggable GC library
- G: Not gung-ho rushing into it for our own use
- G: licensing issues, but that may be addressable
- G: mostly wanted to bring it up, has potential to save some time
- G: contains a mostly copying collector, for example
- G: so if you care to, go read through the docs and give feedback as to whether it's worth investigating the licensing

## Metrics

- G: Just got our first successful metrics collection.
- G: If you write a #[bench] benchmark, the numbers are archived per slave in a JSON file.
- G: In some crates, the results are "ratcheting"
- G: In particular, code gen tests are doing that now, which means that regressions represent build failures
- G: Please do add benchmarks
- G: I have written two benchmarks with "symmetric" ops that are supposed to be the same but one is 10x smaller
- G: Lots of examples in std and extra
- bblum: How much of a regression are we talking?
- G: Currently only the codegen tests ratchet, which is not timing sensitive. It's a strict ratio of our codegen size to clang.
- G: If we were to turn on ratcheting for the other tests, it'd be prob be 10-15% ratio to start
- G: Default behavior is to ratchet based on the noise window
- G: But it's very hard to get a clear signal, we'll have to see how that works in practice
- brson: Is there somewhere we can go to see collected metrics?
- G: Don't have visualizations yet but maybe someone else will pick it up
- G: In the meantime when you run the benchmarks you get these JSON reports
- G: It'll say which ones regressesed and which ones didn't etc
- pnkfelix: Are you planning to extend bors to allow reviewer to override the ratchet check?
- G: Yes though I don't have it yet, I didn't think that there would be a need for it with the codegen tests yet
- G: It'll probably be a build property that you set like any other property
- G: Currently it's a configure flag

## Small code compile time

- G: I am thinking of looking into this. Will I be stepping on any toes?
- *no*

## Unsafe on extern fns

- P: Currently you can write `unsafe` on an extern fn, but they are always considered unsafe, so strcat and I were suggesting perhaps we should just forbid the redundant keyword.
- *crickets*

## Self in impls

- P: Currently `Self` is disallowed in impls. It would be nice to be able to use it, makes it easier to copy-and-paste trait signatures, I propose allowing `Self` in impls and having it resolve to the type that we are impl'ing on.
- brson: Just more stuff, small amount of scope creep, doesn't really matter but I haven't heard this as a big complaint
- P: We can always just defer it, it's BC
- G: I'd leave it as a long term thing, I have run into it a few times
- brson: I'm really worried about scope creep these days, lots of features that keep pushing bar back further
- P: OK
- bblum: I do think it's a good idea, it'd be nice, but post 1.0 is fine
- Sully: There is also something weird about it to me
- N: I've been trying to think if there are complications, I don't think so, but it's a bit odd in that `Self` is a *parameter* in a trait but in an impl it'd be more like a macro
- P: Defer

## self by value in drop

- P: We discussed this before
- P: Two proposals:
  - make self parameter in drop by value
    - most important thing this enables is moving out of self
    - also allows mutating self
  - allow direct calling of finalize trait
    - seems harmless if it's by value, since by value requires you to surrender the reference
    - eliminates an "ad-hoc" rule from the language
- bblum: "Sugar" for `let _ = ...`? (except, of course, you can't call it on things w/o destructors)
- brson: With the by-value finalizer, you need some new rule to either prevent finalization from happening multiple times
- N: Last time we talked about this, we proposed a rule that you could not move self as a whole
- brson: which would also prevent you from putting it in a mutable location
- bblum: what is prob?
- P: could run the dtor more than once
- brson: What are the alternatives? A plan?
- P: We'd also have to relax the rule that you can't destructure things that have destructors
- brson: Trading ad-hoc rules for one another
- P: Yes, but right now a lot of destructors are unsafe, it'd be nice to address this
- P: And we still haven't solved the mutability problem
- N: You can destructure into mutable local variables
- bblum: Why can't you just do "let mut x = self"?
- N: Because then it would be hard for liveness to know that x shouldn't have a destructor run.
- N: I think liveness can handle the idea that `self` is special
- N: But if you do `let mut x = self` then `x` would have to be special too and I don't know how that works
- G: This seems to be in principle a good idea but we don't quite have the restrictions right yet
- P: Maybe we require that you destructure the item in the destructor?
- G: I could imagine that rule, but let's write it up, perhaps it's ok if dtor means you are responsible for terminating the fields

## *mut

- P: General feeling that *mut is not pulling its weight
- P: People write *T in their FFIs and then have to do a lot of transmuting
- P: Because Rust's rules about immutability are not the same as in C, hard to determine what the correct types ought to be?
- P: Probably most every pointer should be *mut except when there are `restrict` qualifiers
- P: Proposal was just to remove `*mut` and make `*` always mutable
- brson: I agree in principle that the diff between * and *mut is basically annotation convenience
- brson: But this makes it impossible to claim that mutability is *always* indicated by the mut keyword, but that would be weaker
- P: But only in the unsafe sublanguage
- N: It doesn't quite hold in unsafe sublanguage anyhow
- kmc: You could presumably pass the * to a C function 
- G: I like the existence of *mut in that * vs *mut is similar to *const
- G: Many C APIs distinguish strongly between in and out params
- G: Most libc arguments *are* read-only
- G: There's something about *mut as documentation that I find appealing
- G: Are we running into a lot of APIs that we are not mapping properly?
- G: I guess bindgen is on its own trying to figure this out
- N: I think bindgen just puts * all the time
- P: I imagine that that's the source of a lot of the problems
- G: I don't have a super strong feeling about it
- P: It is true that this serves as docs
- N: But if you have wrappers, the wrappers would still serve as docs
- P: I think I'm still in favor, although graydon made me less gung ho
- G: In C, for example `strstr` is `const char*`, `const char*`
- N: Presumably the appropriate translation into Rust would be *const char
- ?: Though strstr returns a `char*`, so it's doing an implicit const cast
- G: I think I'm mildly opposed
- N: No strong feeling
- brson: No strong feeling

## Mutable captures

- P: It's kind of annoying that we can't capture mutable variables
- P: Was there a semantic problem?
- N: The original intent of the rule was to avoid confusion in copying vs non-copying
- N: Since any changes to the original value would not be reflected in the captured value
- P: This only affects ~fn/@fn?
- N: No rule against &fn().
- P: This is obsolete with Thunks, right?
- B: Will there be a mutability qualifier in Thunk capture clauses?
- N: I don't know, but as a workaround it would always be possible to do `let mut x = x`
- bblum: There was someone on IRC who came in and was confused because value wouldn't change between invocations
- bblum: I couldn't see how it would work with thunks, unless you stored the entire thunk in a mutable slot?
- N: The thunk takes it arguments by value
- bblum: Only once thunks?
- N: To me a thunk *was* a once thunk, but if there were other traits that are being used to emulate closures, they would take &mut self
- N: Non-once thunks could be stored in mutable slots and take &mut self
- B: Common use case in task code: Capturing a pipe in a task spawn closure and sending on it. This only works today because our pipe (== stream, not oneshot) methods incorrectly take &self (not &mut self), but to fix that, we don't want it to require moving the pipe around explicitly into a mutable slot.
- N: If we have a capture clause, it'd be easier to write mut in there, but maybe that's too much boiler plate
- G: I feel like a lot of these isses are blocked on not having a prototype
- G: Had someone post a tutorial on Rust closures, but if we're deprecating them, we should get on with it
- P: On my list, I've been working on removing @fn, which involves rewriting modules that use the visitor
- G: If you need extra hands, 
- pnkfelix: I've done some work on this as well, we should discuss
- P: OK, is there more to discuss?
- G: Not ready to discuss yet, this is an ergonomic issue, question of whether it will be tolerable

## &const

- P: I've been removing a lot of &const actually as part of the clone work
- P: &const is definitely on its last legs
- P: There are a few remaining functions that use it, but...
- P: I had literally no problems changing the &const to &
- P: So I feel like... it may be time to remove it altogether?
- N: We will have to use something like &const in the borrow checker, possibly when desugaring closures, do we care about that? If the compiler has access to something that the user doesn't have
- pnkfelix: It seems useful for writing tests? Could we use a lint
- N: I like the idea that closures are sugar for something you could express otherwise; this would eliminate that, it may also be useful in FFIs?
- N: Linting it off does seem like a compromise
- pnkfelix: If the big concern is the keyword, can we just solve that?
- cmr (from chat): Some people in IRC have used &const to safely alias &mut
- N: That is exactly the case that I'm talking about
- G: Let me ask a slightly different question. If concept needs to exist inside the compiler, which it sounds like it does, is there a risk to us to turning it off or setting it to a deny-by-default lint? Is there a maintenance burden to keeping it around?
- N: I don't think there's a maintenance burden, the main burden is guiding people not to use it
- G: I agree there's a cognitive burden but if we're uncertain, and we don't think the compiler is going to be vastly simplified, we can leave it "off by default" and relatively undocumented and see if life is OK without it
- G: If we keep finding that we have to turn it on very often, we can go from there
- P: I don't like keeping the keyword around
- G: We don't have a way of linting off keywords at the moment
- G: Though we could use a -Z flag?
- P: I don't know, people will start using the word `const` for various things
- G: We do have reserved words though
- N: What's that?
- G: Keywords we don't use for anything
- N: It's not clear to me that it's a good keyword in any case
- G: One of those C keywords with many meanings
- G: So plan is that we add a lint flag but document it as a reserved word

## for in loop

- P: Mostly a syntax bikeshed.
- G: Difference is existing syntax vs `for x in y`?
- P: brson and I feel like the existing syntax is confusing since it looks like a closure
- P: But maybe we should defer this because we lack time

## Library naming

- bblum: It was pointed out that `get` vs `unwrap` are completely the same
- bblum: In general we are not very consistent about ref vs consume etc
- brson: I think kimundi has a big issue on this #7887
- bblum: I was conflicted, for example, because for ARC we have get vs unwrap, and unwrap is not to be taken lightly, since it does a lot of sync, so in that case I feel that the unwrap operation should have a "heavier" word
- bblum: But in the case of Option, I felt the reverse
- G: Seems like we should chime in on the bug
- brson: I'm glad that people are working on it, it's been a wart for a long time
- N: We'll have to accept that in some cases the names won't be optimal, consistency seems most important
- G: OK, let's all spend some time on the bug today and try to come up with something
- G: Listed as RFC, let's all "C"
- brson: Maybe a motivated person can try to get this to some form of consensus and then mailing list it
