## Attending

Azita, John, Tim, Brian, Dave, Graydon, Patrick, Niko

## Agenda

  * 0.6
  * 1.0
  * any timing constraints, or goals

## Goals

  * **Dave**: my goals for 2013 are for Rust to reach 1.0, let people start using it in production, and for Servo to be able to build momentum
  * **Patrick**: I think we'll end up having to cut stuff; as long as we have a story for adding stuff in the future
  * **Dave**: questions about things that get cut for 1.0 but not cut entirely; move to branch? compiler flag?
  * **Patrick**: yeah, we have some stuff for that now

## 0.6

  * **Graydon**: release cadence has been ~every 3 months; we're talking end of march
  * **Patrick**: yes, sounds good
  * **Dave**: go around room and say priorities?
  * **Patrick**: finish language and improve compiler perf; already done some, graydon doing good work on compiler perf; don't want to spend enormous amount of time on it
  * **Patrick**: another thing: finish all language features that are not experimental
    * resolve stuff that's going to land
    * syntactic changes under discussion (e.g., `impl tr for ty`)
    * INHTWAMA, aka write barriers
    * region syntactic changes
  * **Patrick**: IOW, all breaking stuff
  * **Patrick**: next, combination of language features that can be added later and compiler performance
  * **Niko**: is that stuff you want to do yourself?
  * **Patrick**: yeah, basically. I won't insist I do it myself; people can grab stuff, but that's what I see myself working on in terms of priority
  * **Niko**: won't have a lot of time this cycle. I would like to spend what time I do have on function reform
  * **Patrick**: if it gets close to the end and not done, can you offload it on me?
  * **Patrick**: I would like 0.6 to be soft-backwards-compatible
  * **Niko**: understand
  * **John**: I've been here two days! but I love the sound of my own voice, so... we have a roadmap for hygiene for syntax extensions / macros, and it sounds like first couple steps are:
    * tokentree -> tokentree syntax extension facility
    * then introducing almost a library for hygiene, which is a set of operations for token trees
  * **John**: next three months, no idea
  * **Tim**: my stuff:
    * fixing trait-related bugs, finishing default methods and supertraits
    * would like to keep pushing traits into better traits
    * fixing ICEs
    * pattern reform -- two parts
      * implementation work
      * syntax and implementation
  * **Patrick**: are they breaking changes?
  * **Tim**: no, don't think so
  * **Patrick**: just want to front-load all visible language changes
  * **Tim**: what about refcounting?
  * **Patrick**: we can talk about that; not something that breaks language, but good to do early -- for perf, memory model has big impact so good to front load
  * **Dave**: my gut is saying that language changes *and* GC is a big chunk for one point release
<some mild nods>
  * **Niko**: you reminded me: I had a different approach to cleanups. one of the big unfinished items for regions, I have a design in mind. wouldn't be averse to hacking on that if I have time
  * **Brian**: I intend to work largely on servo, at least 50%. in Rust:
    * state of trait inheritance is bad. would like to fix that in 0.6, whether that's Tim or me
    * not all that concerned about language features. would like to work more on things that help Servo
      * build systems
      * CI
      * etc
    * would like to do more cleanup, completion -- so much garbage all over the place that we need to remove from codebase
    * libuv -- still want to upgrade libuv (going on 8, 9 months now); long-term goal of mine
  * **Patrick**: keep in mind there's a discussion of whether we want to use it at all
  * **Brian**: yeah, probably don't want to use for Servo. haven't looked at NSPR yet, but may end up using that instead of libuv
  * **Dave**: could still be in Rust
  * **Patrick**: but not core; don't want to ship two in Servo
  * **Niko**: don't think we should ship a networking library in core
  * **Graydon**: old argument, ever since we debated with Rafael
  * **Graydon**: this language talks about tasks/events/concurrency more than most at this level; scheduling is very tied in. I'm very nervous about untangling them. worried about multiple event loops fighting with each other
  * **Graydon**: really hesitant about that line of argument, but we don't have to talk about it today
  * **Graydon**: don't understand how NSPR could be used in any production codebase
  * **Patrick**: have to for NSS
  * **Graydon**: this is not the meeting for that debate
  * **Graydon**: I'm next in line
  * **Graydon**: few that I'll try for:
    * GC
    * fixing unwind system
    * speed of build system / slowly getting away from makefiles
  * **Graydon**: worried about driving contributors away due to being too slow to build
  * **Dave**: do we need to land LLVM changes upstream?
  * **Graydon**: don't think we need to, getting simpler with GC and refcounting so backing away from needing to do that
  * **Patrick**: agreed
  * **Dave**: so that sounds like a good 0.6 goal: get back to using main LLVM, whether that requires landing upstream changes to them or not
  * **Patrick**: all we need to do is diff since last branch point
  * **Graydon**: more concerned about code that supports it bitrotting; if we get out of tree, wanna be sure we keep it out of tree
  * **Graydon**: we'll put that as a stretch goal for me
  * **Graydon**: so is all that good?
  * **Dave**: does it all seem realistic?

Time:

  * Patrick: 80 - 100%
  * Brian: 50%
  * Niko: 20%
  * John, Tim, Graydon: 100%

## 1.0

  * **Dave**: I would like to feel confident that our memory management system is robust and performant enough for 1.0
  * **Dave**: It doesn't have to be perfect, but I would like it to be... stable.
  * **Dave**: To get more specific, I want it to be the rough architecture that we believe in and I want it to have a performance baseline that is not an absolute dealbreaker for customers
  * **Dave**: Other major thing is overall library design, which we have not even begun to address
  * **Dave**: A lot of open questions about how it should be organized, what are the main traits that we want, which batteries should be included
  * **Dave**: I'm beginning to think that having enough libraries out of the box is very important for language adoption
  * **Dave**: I think there is a lot of work to do there and I don't know how we want to break it down, it doesn't seem like something that we can just divide amongst people, requires some central coordination
  * **Dave**: Maybe that starts with 0.7? I don't think it's 0.6.
  * **Patrick**: To add to that, there are big perf. issues with the libraries right now.  I/O for example is inefficient by design.
  * **Niko**: sounds like we need a "best practices for performant I/O..." --
  * **Graydon**: problem with a lot of our libs is they were designed a long time ago
  * **Niko**: yeah, they're forcing allocation along paths that shouldn't, etc...
  * **Niko**: seems like we should have some policies
  * **Patrick**: I'd vote "no gc in libstd at all"
  * **Niko**: that's probably hard. persistent functional data structures, for example. maybe not in libstd, but anyway should be a high bar, yes
  * **Patrick**: good overarching this for 1.0: there's a community interested that will leave if they see any overhead over C++
  * **Dave**: could still draw a dividing line and call some "higher level libs" or something, but point taken
  * **Patrick**: package management and documentation
  * **Patrick**: some sort of build system, doesn't need to be full fbuild workalike, but something
  * **Patrick**: need something like `cargo install` to work for binary packages -- stuff that has C/C++ components -- `cargo build gmp` and if we don't have it installed it goes and builds it
  * **Patrick**: basically, nice docs
  * **Patrick**: in terms of performance: do well on kinds of things people will immediately compare to other languages for. for example, cdleary's printf test should do well. spawn 1000 tasks compared to 1000 actors in erlang, for example.
  * **Patrick**: go over stdlibs and make sure they conform to reasonable style we expect
  * **Patrick**: need to have enough language features in that you don't stub toes on something, e.g. one-shot closures needing `Cell`
  * **Dave**: does 0.6 do most of that?
  * **Patrick**: front-loads breaking changes. biggest toe-stubbing is write barriers and one-shot closures.
  * **Patrick**: other things like macros not being scoped
  * **Patrick**: don't mind if you hit limitations of trait system -- people hit limitations even in Haskell
  * **Dave**: docs is a good point, though: we'll need a full language manual
  * **Graydon**: wanna make a plea: language grammar is ill-defined. we don't know what class parser is in, don't know it's deterministic, ...!
  * **John**: would love to do a little fuzzing, would love to work on that
  * **Graydon**: would like our docs to have grammar rules -- they did once upon a time. would like to test our parser against those rules
  * **John**: happy to help, sounds like fun
  * **Graydon**: huge amount of technical debt. library redesign is true. but tons of support code in compiler for obsolete features that don't even make any sense. not keen on 1.0 continuing to have support for deprecated things. want to get that cleaned out.
  * **Patrick**: we're not that far now; I have a patch that removes a bunch
  * **Graydon**: I know, we're going step by step
  * **Patrick**: get all legacy features out of rustc
  * **Graydon**: we have a redundant COM library in there...
  * **Niko**: I think it would be great if rustdoc produced a web page and not markdown
  * **Niko**: could we get some help with that?
  * **Dave**: quite possibly. we'll see what we can get
  * **Dave**: all you really mean is generating better web page, not whether it passes through markdown
  * **Patrick**: can we get resources to do like a Rust playground?
  * **Dave**: good point, shouldn't be done by this group, but we'll see what resources we can get
  * **Graydon**: package management and build system. lots of versioning questions. language versioning and symbol versioning. not hard but will take a solid month or two to iron out.
  * **Graydon**: performance-wise, I agree we should have not-embarrassing performance out the gate. we ported some benchmarks but had trouble with idiomatic translations. should have sensible translations
  * **Patrick**: benchmarks currently are Rust 0.2 that have been kept 0.2
  * **Dave**: idea of including performance tests in CI so regressions are caught immediately
  * **Graydon**: yes, had trouble getting that going before, but some ideas for doing it
  * **Graydon**: we're definitely not catching perf regressions right now; have been decaying performance with feature work. which is fine, but we'll have to stop
  * **Graydon**: anyone have language changes that are big semantic changes, not scoped for immediate future, that they'd like to include for 1.0?
  * **Patrick**: hang on, will be right back with a picture of my board
  * **Patrick**: here's my list:
    * cfg: conjunction (can do disjunction) and negation
    * impl trait for type syntax (needs discussion)
    * constants and phase distinction
    * fixed-length vector with constant length (constants can themselves have types in them, so tricky) -- easiest thing is maybe removing field-projection in constant-evaluator; dependency graph only ... well, we can discuss later
    * method reform: ability to call methods as functions
    * don't accept unenforced bounds: some bounds problematic to enforce
    * trait method privacy: have to figure out what `priv` means in traits
    * `Ord` trait makes it impossible to implement efficiently
    * warn unused result: nice lint mode
    * unsafe pointer indexing
    * `&alias mut`
    * C function reform: need to reform extern functions; stack switching, etc needs unified story
    * methods not accessible under both type and trait
    * region syntax
  * **Dave**: what about runtime-less Rust?
  * **Patrick**: needs to be decided for 1.0
  * **Brian**: impacts design of core
  * **Patrick**: impacts a number of things
  * **Dave**: need decision and direction, not necessarily needed to release with 1.0?
  * **Patrick**: would really like to ship with it, but won't necessarily block 1.0 on it
  * **Graydon**: I always thought your whiteboard would be more terrifying, I'm not so scared of that
  * **Niko**: wanted to say: there was the discussion relating to single inheritance etc for Servo; think that's not hard to implement, but need to discuss
  * **Patrick**: not even sure that has to be in 1.0
  * **Patrick**: needed for Servo perf, but maybe not needed for users
  * **Dave**: could potentially be good to leave out for helping people understand it's not central to Rust
  * **Graydon**: so that all sounds reasonable. anything scaring anyone?
  * **Dave**: the versioning stuff maybe looks the most opaque but I believe you'll get it worked out
  * **Graydon**: not that opaque in my mind
  * **Patrick**: libuv vs NSPR needs to be worked out
  * **Azita**: talking end of 2013?
  * **Dave**: yes, 5 point release would roughly land us at 1.0 by end of year. this is just back of envelope but feels both healthily aggressive but not unrealistic
