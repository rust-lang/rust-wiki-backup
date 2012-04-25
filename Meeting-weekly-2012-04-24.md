## Attending

Graydon, Marijn, Tim, Patrick, Brian, Niko, Dave

## Graydon updates

  * **Graydon**: working on getting crashes out of slices; starting to work pretty well! still some random crashes, doesn't seem to help cycle time that much
  * **Niko**: no massive improvement after doing I/O lib?
  * **Graydon**: nothing I can see, but I think my benchmarks aren't perfectly honest; focused on crashes for now; I'll do more benchmarks
  * **Patrick**: I wouldn't expect massive increases; one of us would've made it unsafe by now if it were taking a larger portion of our time
  * **Niko**: there are a lot of other vectors outside our uses
  * **Graydon**: yeah
  * **Marijn**: still waiting for writeup on public/private
  * **Patrick**: I had a small writeup... I will send to the list
  * **Graydon**: hoping this month is the last major syntax spasm
  * **Niko**: hope so
  * **Graydon**: I appreciate we want to address ugliness; but at some point people are gonna start not taking it seriously they can write anything in the language because the syntax is changing; but everyone knows that tension is there
  * **Patrick**: I'm personally going to resolve to not talk about syntax for a long time

## Patrick update

  * **Patrick**: I've been working on GC infrastructure in LLVM; not just GC but how we will do unwinding; that frees us from making landing pads; should reduce our code size and work on Windows; infrastructure same: stack map and stack crawler can read stack map and do things to variables in it; involves pretty invasive changes to LLVM; done a few prototypes, started on what I think is cleanest version yesterday: tag pointers with address space; boxes space 1, uniques space 2; uniques self-describing; enums need to use llvm.gcroot intrinsics, but don't need that if LLVM type is sufficient to find all pointers (i.e., anything other than enum); I have a prototype of that stack map info working; redoing not to require any intrinsics at all; half of that done yesterday; will probably have some fallout so I'll bring back --gc switch
  * **Brian**: we're going for unwinding first, right?
  * **Patrick**: have to make uniques self-describing to do that b/c LLVM type info isn't enough to recreate the shape, so have to do that first which would be a distraction; so I'm focusing on boxes right now since they're already self-describing; don't plan on landing and making us go to full tracing GC yet; not immediate pressing concern for 0.3; higher priority is unwinding on Windows and getting rid of landing pads
  * **Patrick**: gonna try to get rafael to review to get upstream to LLVM
  * **Dave**: Chris Lattner visit: can discuss with him
  * **Patrick**: hopefully will be working by then and he could take a look; it's not *super*-invasive, but does touch a dozen or so passes in LLVM
  * **Niko**: cool! that's excellent
  * **Brian**: Graydon, when do you intend to pull in clang to our repo?
  * **Graydon**: already did a test of that, have a branch in my private repo where it's buildable; haven't gotten further; next in line on my todo list after slices work; near-term
  * **Brian**: not blocked on it, but many interesting things can happen at that point
  * **Graydon**: yeah, like a dozen bugs downstream of it
  * **Patrick**: do we want to talk about mac bundles? maybe add ability to make mac bundles to rustc or as a plugin?
  * **Brian**: think it should be a lib, a tool in our growing suite of rust tools; eventually all stuff will be pluggable
  * **Patrick**: do want it to be part of a build process
  * **Dave**: once it's pluggable, could be a plugin
  * **Niko**: could be a wrapper around rustc
  * **Patrick**: well, larger questions here about how we want rustc to be part of build processes
  * **Brian**: think we're gonna want it both ways; don't have capability to do that yet
  * **Patrick**: Graydon, thoughts?
  * **Graydon**: I haven't run into a hard and fast rule; I've been thinking of a top-level rust command simply to provide an end to the drivers-that-drive-drivers-that-drive-drivers madness; not necessarily a subtree in that particular tree; whether rustdoc is a separate tool or part of the top-level tool...; these are aesthetic questions to some degree; look at gcc -- it does both
  * **Patrick**: rust-bundle is kind of adding a driver that drives a driver
  * **Brian**: so we'll have this high-level rust tool...
  * **Dave**: would rustc go away then?
  * **Patrick**: well, separate binaries
  * **Brian**: well I like LLVM approach as libs instead of binaries
  * **Niko**: that's more a matter of how we expose our tools; could still allow plugging separate binaries
  * **Graydon**: interfacing question: to what extend to we allow interfacing external bins vs internal libs; I don't have clear aesthetic preferences yet; worth discussing on list, polling for opinions and experiences on
  * **Graydon**: we've come up with ad-hoc guidelines for other aspects of UI (e.g., "if useful for part of a build, should be expressible as an attribute") -- should come up with guidelines for build process as well
  * **Brian**: Patrick, in favor of starting new crate for macos bundles?
  * **Patrick**: I don't have an opinion, just laying out some constraints; I'm in favor of anything that solves the issue of making Mac bundles
  * **Graydon**: there'll be Android package maker, etc etc etc -- Mac bundles won't be the last of these issues
  * **Graydon**: there are abstraction issues about running subprocesses -- not always possible to do portably; have to think about this more; I'll poll the list

## Brian update

  * **Brian**: been doing a lot of work trying to get gfx working for servo; lot of things related to bindings; Servo using bindings for SDL, Cairo, Cocoa, xlib, and Azure; using all them and can display graphics now; some fed back into Rust, but mostly working outside Rust repo
  * **Brian**: did a lot of work on bindgen
  * **Brian**: inside Rust, working on making LLVM assembly a little more readable; patch that makes enums into LLVM named structs; done but perf regression so I have to investigate why
  * **Brian**: refactoring rustc a little; split out syntax crate, trying to clean up parser and lexer to make them look more like the description in the manual so we can start testing the grammar
  * **Brian**: also been looking at metadata module in Rust: would like to get dynamic loading working eventually; gonna need most of the metadata module available to do dynamic loading safely
  * **Brian**: trying to figure out how to get rid of as many dependencies as possible; what I think will end up happening: all of ty will have to go with all of metadata; really tightly coupled, will have to go into one crate together; everything else we can probably separate
  * **Brian**: metadata depends on syntax; in order to dynamic loading at least in near-term, will need all of syntax crate and metadata crate
  * **Niko**: any thoughts on how to make metadata less horrible?
  * **Brian**: no, not yet

## Niko status update

  * **Niko**: working on regions, or "lifetimes" as Patrick and I think it should be called
  * **Niko**: coming along well; pieces falling into place
  * **Niko**: one unresolved question about how to handle function types but I think it should be fine
  * **Niko**: still a few more bugs left to close; major thing needed is iface/impl stuff: have to make that work (think I may have broken it)
  * **Niko**: otherwise in pretty ok shape; gonna start making more test cases
  * **Niko**: you can look at the open issues
  * **Graydon**: nomenclature question: has the term "lifetimes" been used in the literature?
  * **Niko**: I think "regions" is the wrong intuition
  * **Dave**: can use "regions" with research community and "lifetimes" with users
  * **Graydon**: just want to make sure code switching doesn't screw with our heads
  * **Niko**: hopefully we can experiment soon; was trying to write up a blog post that introduces some patterns and how I think they should work; maybe y'all can comment
  * **Niko**: still in a trial period; maybe we wanna tweak e.g. syntax before we unleash it
  * **Graydon**: I suggest a soft launch; give it a try on your own inside the compiler in a private branch; try rewriting some code and see how it goes

## Marijn status update

  * **Marijn**: rewriting pattern matching, exhaustiveness checking based on Ocaml's
  * **Marijn**: I have to move on to resolve, haven't started yet

## Tim status update

  * **Tim**: still plugging away on classes
  * **Tim**: working on casting classes to interfaces this past week; tried to leave some comments behind me
  * **Tim**: did some refactoring
  * **Tim**: list of about 10 more to-dos on classes; should I close the classes bug and make separate specific bugs?
  * **Dave**: do we do tracking bugs?
  * **Graydon**: no dependencies in github
  * **Tim**: also implemented star syntax for bind all fields in a variant; when there's a new snapshot, people can make code slightly smaller as they go along
  * **Niko**: I think making individual bugs is good
  * **Graydon**: can up-and-down prioritize, too
