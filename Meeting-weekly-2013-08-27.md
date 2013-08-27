# Agenda

* rustpkg timeline for completion (tjc)
* extern fn "checkin" (nmatsakis)
* generic extern fns
* llvm assertions
* Local::<for Trait>
* "unsized" prefix / Sized bounds
* zeroing out, no drop flags
* patterns in default methods
* macro rules behind a flag
* == auto-deref
* Index traits
* overloadable deref
* @ patterns

# Attending

ecr, dherman, azita, tjc, brson, pnkfelix, nmatsakis, jack, lbergstrom, kmc

## Rustpkg timeline

https://github.com/mozilla/rust/wiki/Rustpkg-schedule

- tjc: i wanted to talk about the schedule
- tjc: wanted to include deadlines, wanted to know if they're realistic. started pulling out needed bugs, and started working w/ jack to port servo to rustpkg. most important bugs I put at beginning under "ready for use" -- once those are fixed, work on servo with rustpkg can really get started
- tjc: used same milestones we have for rust. one w/ most bugs is "feature-complete". left about 10 bugs that are more like enhancements.
- tjc: what I'm calling production-ready I'm putting at 2/7. want people to look this over and see if it's realistic
- jack: I went thru servo deps this morning and see which bugs being fixed would enable which crate
- jack: 8405 and 7240: not sure what those are
- tjc: semantic versions means supporting semver, which may not really be needed for rustpkg. graydon thought it was important but I didn't follow all the reasoning
- tjc: multicrate pkgs means a single pkg w/ more than one crate. also something graydon thought was a priority but maybe not as needed for servo
- jack: single directory contains multiple lib files?
- tjc: not multiple lib files, but single pkg w/ multiple subdirs each w/ its own main or lib files
- jack: ok. not sure how that differs from case w/ multiple pkgs in same workspace
- jack: don't think either of those two needed for servo. might still be worth having for other reasons
- jack: other ones that weren't on this first list that we'd probably need are: 8520 -- don't think we need that if we have 6408 (subdirectory stuff). 7402 we do need but it's already in the top list
- jack: we could build 11 crates if this top list of bugs was done
- jack: includes some platform stuff... all crates that either have no dependencies and are pure rust or only depend on system libs (and each other, of course -- which need 8524, recursive deps)
- jack: after that, rustcocoa, rustglut, etc are pkgs w/ only C deps. if we could compile a single C file and link it into the crate that'd give us 3 or 4 more crates. all remaining crates in servo require ability to compile a C library or be able to use a C lib we compile some other way. libpng etc.
- jack: think importance of passing RUST_CFLAGS to rustpkg (8522) -- if we move that farther up we could get servo on rustpkg sooner
- tjc: that makes sense
- jack: currently due 12/16
- tjc: imagine way building C libs would work at first: custom build script: package.rs file that shells out to make or whatever. at beginning could just add link flags from build script, which is part of 8522.
- brson: is flags we need primarily -L?
- jack: AFAIK only -L.
- jack: might be something in Android stuff but pretty sure it's just -L
- jack: so if we had this first list of stuff + 8522 we could get all of servo into rustpkg
- brson: does seem primary goal is to port servo
- tjc: and eventually rustc
- brson: so new opinions change timeline some?
- tjc: sure, after meeting I'll update the timeline. that's helpful. wanted to make sure people think these are reasonable deadlines, not too ambitious nor too far out
- felix: jack, you made distinction between single C deps and general C deps. is that distinction worth making?
- jack: difference is for single C file just invoking gcc; for others we need configure/make
- jack: this will get things building with rustpkg -- oh, forgot one. need to manually specify dependencies. need to say that we depend on a C file so if the C file changes we have to rebuild. that wasn't on the list at all, because tjc and I just talked about it yesterday
- brson: sounds pretty difficult
- jack: not automatic, just need to be able to manually add something to dependency list. it can be super simple. but if we got this building and we didn't have those deps, then if somebody changes rust-azure you'll have to manually rebuild, which might be ok
- dherman: well, automation is good. seems like a footgun not to have it
- brson: jack, will you be porting servo to rustpkg?
- jack: we'll get rid of makefiles, top-level makefile will just invoke rustpkg with appropriate RUST_PATH. will get rid of 11 different pieces of our build system which is nice
- jack: as rustpkg gets better we'd continue expanding number of crates
- brson: so december is generally a dead zone and jan 15th has a pretty major milestone
- tjc: that's true, trying to front-load what it needs but I could move it out a bit, move it to 1/30 and move other things accordingly
- brson: "rewrite test.mk in Rust"?
- tjc: that's sort of part of prior work items
- brson: are you thinking of doing that?
- tjc: not necessarily, I could do it or someone else
- brson: I think that's a huge effort
- tjc: maybe that shouldn't be on same milestone
- brson: probably. rustpkg will initially be a small subset of what test.mk does
- tjc: want to capture what's needed for calling rustpkg "complete"
- dherman: yeah, "complete" is really the first iteration, it can grow functionality over time
- jack: one of the first libs we can use with it is opengl-ES -- once those kinds of things are being built with rustpkg, people will start finding bugs and community contributions will pick up eventually
- tjc: yeah, that'd be great. pretty soon I should email the mailing list and invite people to contribute. it's getting to a point where there's enough infrastructure they won't necessarily get lost or rathole on architecture
- tjc: that's great feedback. there are questions we didn't get to in last week's meeting but I can incrementally bring those to this meeting
- dherman: sounds good

## extern fn "checkin"

- niko: some aspects of extern function I took on, wanted to run them past group
- brson: I don't quite understand the extern fun macro, what it does exactly
- niko think it needs a little work. rest of design makes sense: when you call a C function we don't do anything special with stack, just call it. but there's a lint that the Rust function that does call has an annotation to have a big stack. you can put those further up the call chain if you prefer
- brson: do you have to annotate all chains?
- niko: you just to do highest one
- niko: lint will complain if you don't do the topmost frame. if you prefer somewhere further down the stack you can put it anywhere
- dherman: i don't annotate things at runtime. what do you mean?
- niko: i mean you need to annotate the functions
- dherman: are we talking about whole program analysis? ...
- niko: right. once you switch to a big stack you never switch to a smaller one. you annotate the point at which you want to switch
- pcwalton: lint mode is *not* a sophisticated static analysis
- niko: not sophisticated, no, just a lint mode. immediate caller of C function must be annotated
- niko: meaning of annotation is "upon entry to this function, switch to a big stack"
- dherman: dynamic check to see if you're always on a big stack?
- niko: yes, but same dynamic check we already do for stack overflow
- tjc: don't understand something: patrick says correctness of program is dependent on annotation being correct, but if you satisfy the analysis it guarantees correctness.
- niko: if you use default settings, that's true
- tjc: oh so you could turn off the lint mode
- niko: yeah, it could crash or segfault but it'd be through an unsafe section
- niko: that can actually happen no matter what. to extend we use system stacks, we can use same techniques as C programs like guard pages
- dherman: I like that this is "Rusty" in giving you control over perf, but avoids complexity of effect system
- niko: but you can disable annotations on a 64-bit system by turning off lint mode, or if you know your program only uses big stacks
- brson: the name "fixed-stack segments" exposes our impl details
- niko: yeah name isn't that good. going forward would like something like "required stack size"
- brson: I'd like taht too
- niko: oh I never answered your question. the extern fn macro creates a function and a wrapper that adds all required annotations. basically does what we used to do
- brson: does it declare an entire extern block?
- niko: yes, local to the function. might eventually want the macro to be able to define multiple functions together
- brson: what about linkage annotations?
- niko: you can put annotations in there
- niko: main thing we'd wanna change is be able to accept multiple ones, e.g. there are some platforms where you have to define them in groups, and this doesn't handle that very well
- niko: macro accepts arbitrary annotations in front and those get placed on the extern block. if you need more control than macro provides you can write it yourself
- brson: this allows you to declare extern funs individually which you couldn't before. kind of always thought you should be able to declare just an extern fun.
- niko: that's kind of what I'm getting at. don't deeply understand the difference of "loading a framework at a time" on some platforms. maybe someone else knows
- brson: I haven't encountered it.
- niko: OS X frameworks came up in previous conversations. not sure. would be something to look into
- brson: don't know if we do anything special when we do that
- felix: talking about dynamic symbol resolution in OS X? or something else?
- niko: don't know what I'm talking about! can anyone else fill in?
- niko: maybe I imagined the whole thing
- pcwalton: you're not imagining it, we're talkinga bout 2-level namespaces
- pcwalton: https://developer.apple.com/library/mac/documentation/developertools/conceptual/MachOTopics/1-Articles/executing_files.html
- pcwalton: the first level of the two-levle namespace is the name of the library that contains the symbol, and the second is the name of the symbol
- pcwalton: With the two-level namespace feature enabled, when the static linker records references to imported symbols
- pcwalton: it records a reference to the name of the library that contains the symbol and the name of the symbol.
- niko: while adding fixed-stack segments I noticed lots of places that were doing this in loops, and this will certainly help them
- jack: if you add multiple places in call chain will it only switch at top?
- niko: that's the idea but mechanism it uses is perhaps not ideal. reason it will (might need some refinement): when you label fixed stack segment it'll request like 2MB but gets more. if you request it twice, your second request will have no effect. but it is possible in between that you'll have used up the extra. could imagine when you have one of these annotations you might want to do it differently, maybe should check how much space has been allocated or something... could emit different assembly, maybe needs a different field... should be something to hammer out
- niko: if we have different categories, maybe we'd do that a little differently
- dherman: don't imagine you want *too* sophisticated machinery, doing all kinds of arithmetic
- niko: imagine you want something that just says "ensure I have this much stack". seems like most flexible to me, but we should probably drive it with use cases.
- brson: I had one other concern about this: this ended up uncovering a new linkage issue
- brson: feel this indicates a broader issue we don't know what's going on with the linker
- niko: to recap for others: now that we're linking to C function, when you have crate A that exports a C function and crate B uses it, linkage error arises. not sure why. maybe because crate A didn't use the function itself and linker stripped it out? not sure
- niko: seems solvable through various mechanisms, like one option is don't allow extern functions to be exported, one option is understand what linker is doing, make it stop with dummy references
- dherman: I'd hack until the linker is happy, shouldn't be *too* hard I'd hope
- niko: just a hypothesis, don't actually know what went wrong
- pcwalton: there is a thing called @llvm.used that might be useful, lets you mark functions not to be stripped
- niko: should try that to see if my hypothesis is correct
- niko: already do a transitive walk to find if you include a crate A include all its deps, maybe should gather up transitive set of C libs
- pcwalton: we do that already b/c it was needed for OpenGL or something (I added it)
- jack: if you have relative paths now they get passed as relative paths to other crate, so may be completely wrong. servo sets those up more correctly but minor miracle it worked
- pcwalton: yeah we should fix the relative paths
- brson: niko, satisfied?
- niko: yeah, I'm not hearing any -1's
- pcwalton: +1 for me on the design

## generic extern fns

- brson: complete disallow except for intrinsics
- nmatsakis: I... thought I did this already. If not, yes, let's ban them.
- tjc: yeah, how do you monomorphize?
- eric: void * I guess
- tjc: what could possible go wrong... ;)
- (general agreement)

## LLVM asserts

- brson: very big portion of LLVM runtime
- tjc: back of the envelope sense of how much we'd save?
- pcwalton: I don't have a number but someone did this, it's an issue somewhere
- tjc: way to turn this off, maybe when rust debug turned off?
- jack: it took 15% off build time. 20+ mins -> 17 mins (reference https://github.com/mozilla/rust/issues/3615)
- niko: I was thinking maybe it's possible to turn them off unless you touched trans
- brson: chuckle
- tjc: concern is a user might be dismayed if they see a segfault, maybe an assertion failure would be more helpful. but maybe not since they thought an LLVM assertion failure meant they had a bug
- niko: helpful to us
- tjc: yeah, but maybe if there's a segfault we could just do it ourselves
- pcwalton: 15% isn't life-changing for bots. nice for users but bots: meh
- jack: jdm mentioned we want this for servo
- keegan: disable LLVM asserts?
- jack: yeah to give faster build speed
- brson: we already have this configure option
- keegan: could be something we turn off in time, but might still be getting value now
- dherman: is this a good time to ask about landing LLVM patches upstream? can we not have to build LLVM as part of Rust? when?
- pcwalton: eventually do need to put an end to fork
- felix: https://github.com/mozilla/rust/issues/4259
- brson: some issues related to split stacks on ARM
- brson: in many cases we're just re-building the exact same LLVM. we could just be using the snapshot
- felix: I thought bots aren't rebuilding on every run
- tjc: we're talking about onboarding experience
- brson: requires more build logic I guess. there are situations where we have a snapshot that's not the one you need
- tjc: could we make a new snapshot every time we change LLVM revision?
- brson: think there's a tiny issue there, but pretty simple to do
- jack: can we snapshot LLVM separately?
- felix: why not prebuilt binaries?
- brson: we could get there, not too hard. at least a week of work
- dherman: very valuable. would love to get to a point where we have prebuilt binaries for the onboarding experience
- brson: we'll have to think about how that impacts build system

## Meeting time

- azita: should we move to 10am?
- dherman: used to be 9am for Marijn who was in Berlin, what about felix?
- felix: 10am is fine
- azita: I'll update the meeting invite
