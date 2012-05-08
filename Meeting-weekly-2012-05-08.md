## Attending

Niko, Patrick, Paul, Tim, Marijn, Graydon, Dave, Brian

## Vector types and dynamic-sized types

  * **Niko**: reactions to what I've described?
  * **Graydon**: two things tied together: 1) syntax proposal; don't like leading square brackets, 2) dynamic size is motived by couple concerns: I don't really see a need for it; what I'm not clear about is whether there's a version of classes that includes subtyping / record extension
  * **Tim**: thought we weren't going to have inheritance
  * **Niko**: I think there's no plan for open subtyping
  * **Patrick**: we've talked about it, and we have a sketch of something we *don't* want to do (we sketched it out, but decided against it); I share your reluctance to move to a traditional OO system
  * **Graydon**: yeah, a degree of brittleness I don't really wanna bake into the system
  * **Graydon**: this seems like it's really just about vectors
  * **Niko**: it can be generalized, for e.g. function types and ifaces
  * **Graydon**: don't understand how undetermined size... you're saying you want to write them with regularized syntax?
  * **Niko**: in practice that's what it amounts to; either you say @fn is some sort of indivisible type
  * **Graydon**: I agree grammatical compositionality would be good to hold on to -- not have special cases for @FOO
  * **Patrick**: I like that @ means only one thing; you could fix compositionality by moving @ around, but I want it just to mean one thing no matter where it occurs in the grammar
  * **Graydon**: don't we have an implicit different type of capture though?
  * **Patrick**: yeah, so capture's where it gets funny
  * **Niko**: I did put up a blog post up but don't expect everyone to have read it; fn:send would be fn that closes over sendable data; proposal separates out the bound on the closed-over data from the allocation type of the function
  * **Patrick**: I like the idea of only having one function type
  * **Dave**: table for now since the blog post is brand new and no proposal yet
  * **Graydon**: I can see this holding together; I appreciate the degree to which orthogonality is part of the design pressure, but it's not quite clear that the result looks/feels nice; have to toy with it a bit
  * **Niko**: I like the idea but I understand your objection; would probably be healthy to write it all out, do some before/after gists
  * **Patrick**: ideally prototype it, but might take a lot of work
  * **Niko**: maybe a day or two, not a whole lot

## Borrow check for regions

  * **Niko**: guarantees that no region pointer is deallocated while you're pointing at it; subsumes alias.rs
  * **Niko**: want to check in w/ the group; what kind of analysis we want to do
  * **Niko**: would like it conceptually if we took conservative assumptions, no type-based analysis; would be nice: if you're going to work with a unique pointer (dangerous b/c it gets deallocated immediately), has to be rooted in the local stack frame; the borrow check can then look just within the function; but then you'd have to swap pointers to the stack when you work with them
  * **Niko**: when I run my borrow check now, there are 46 violations in the existing code
  * **Niko**: a couple options; does anyone strongly object to this idea? should we have a stronger analysis, or can I forge forward?
  * **Brian**: will this get better as Graydon changes vectors?
  * **Niko**: if we're using GC'ed vectors, this doesn't matter b/c we can be more liberal w/ those; same holds for slices
  * **Graydon**: well, you can't append to a slice
  * **Niko**: not for appending, no
  * **Niko**: but other operations, searching etc, are safe
  * **Niko**: other thing: we can encapsulate this growable-vector pattern with a class
  * **Patrick**: I was more interested in these kinds of fixes; would you do this in a library?
  * **Niko**: yeah
  * **Patrick**: it's probably inevitable we'll have growable vectors in the std lib; seems like a natural way to fix it
  * **Niko**: only downside: because these swap to the stack, if you use recursively you get an error
  * **Patrick**: like a concurrent modification exception
  * **Niko**: well, but this is even for read-only access patterns
  * **Patrick**: you could keep the length :)
  * **Graydon**: having trouble understanding the failure modes; this is delicate stuff; let me take a step back and say, could you present in terms of something the current alias checker does check and that you're proposing ceasing to check, and why
  * **Niko**: current alias checker permits some things unsoundly; you can access freed memory; not sure if that's a bug or a deeper problem with the analysis
  * **Patrick**: probably just bugs
  * **Marijn**: not sure how much more complex it would get after we fix the bug; but what you get from type-based alias analysis is that it proves you can't access this value b/c you're not passing anything of this type in; but if we add globals this falls down anyway so we may have to eliminate it anyway
  * **Graydon**: I'd like to clarify that; I don't think we're talking about writable globals
  * **Marijn**: read-only globals?
  * **Graydon**: yeah, effectively, or that can be assumed as such outside of any unsafe code that modifies them
  * **Niko**: I know you wanted const so they're deeply read-only; conceivably user might add a library that doesn't include that kind of restriction, uses a global variable... seems risky to me
  * **Graydon**: this is all fair, but I think it's incumbent on us, if we design a global system, that it has to be safe in its use outside of unsafe blocks; you can of course break the alias analysis and the entire memory model with unsafe code; we can't insist unsafe blocks should be safe
  * **Niko**: what I was saying is it's possible to easily build an abstraction which gives access to data that it shouldn't; maybe. not sure
  * **Niko**: other downsides to alias-based analysis; when it works, it's great; but it's weak about closures; take example of std lib vector: if you're iterating through a vector, great, but suddenly if another vector comes along it can fall down
  * **Marijn**: the alias analysis is already doing the other stuff you're talking about, just uses type-based as a fallback; I was surprised by the number 46, expected more
  * **Niko**: I was a little surprised too, actually, maybe there are bugs in my code
  * **Patrick**: you were saying all of the 46 errors were mutable vectors, and would be solved by a growable vector class?
  * **Niko**: essentially. I think there were a few cases which didn't seem very important to me, e.g. accessing token array in parser by reference, which is mutable, which could be changed
  * **Patrick**: my inclination is to try the dynamically sized class and see if it works; let's use the minimal amount of machinery that works
  * **Niko**: I can push forward and see what seems best to me and then see how people feel
  * **Niko**: I could bring it in and leave it as warnings
  * **Patrick**: it's not particularly risky to bring it in; we could always add the analysis later; doesn't force us down any particular path; adds some restrictions for now, but that's it
  * **Graydon**: this analysis does not apply to the existing modes?
  * **Niko**: it does consider the modes; treats them like borrow
  * **Graydon**: I don't understand the mutable library with swapping trick -- could you sketch that in a gist?
  * **Niko**: I'll send you a link
  * **Graydon**: I'm not opposed to continuing to pursue; it's cool if we can get away with it, but if using types helps, I'm not opposed to it
  * **Niko**: we could also potentially use the pure qualifier
  * **Graydon**: we talked about having a dynamic discriminate check; if you pass this point successfully, these two things do not alias
  * **Graydon**: a little bit of precedent in C with restrict
  * **Dave**: I like that more because it's more clearly stating your intentions
  * **Graydon**: I understand that, yeah, and agree
  * **Niko**: we really should be able to call vec::len everywhere, hence the motivation for pure
  * **Patrick**: doesn't solve vec::each
  * **Niko**: true, it doesn't
  * **Graydon**: would nice to be able to state two regions are disjoint
  * **Niko**: requiring two regions be disjoint is the subject of a lot of research... anyway, I'll send you a link to my blog post

## GC support

  * **Patrick**: kind of misnamed; I don't actually care about GC right now, but I do care about unwinding working on Windows
  * **Patrick**: uses LLVM GC support to make unwinding work on Windows
  * **Patrick**: idea is to get away from using landing pads, instead use accurate stack maps; should result in big code size reduction
  * **Patrick**: status is interesting; Chris Lattner came by last week, chatted about how to make this work; we had an email followup conversation; have an issue filed with my current thinking and status
  * **Patrick**: lots of pieces in various stages of completion; hoping to finish and bring them together into a unified whole over the next few weeks
  * **Patrick**: what I wanted to bring up: fairly fragile, requires tracking information (specifically, point at which values are pointers) through much later stages than LLVM has previously tracked in the past
  * **Patrick**: basically, in rustc we tag boxes as addrspace 1, uniques as addrspace 2; GC root enums
  * **Patrick**: in LLVM, need to track the addrspace through compilation; tricky thing is need to be cautious when it comes to LLVM optimizations -- need to make sure they don't destroy the identity of pointers
  * **Patrick**: safest way to do that is to introduce a GC switch, have that disable all LLVM optimizations, get it working, then re-enable optimizations one by one once we have reasonably good test coverage once we are confident the optimizations aren't breaking things, and fix them whenever they do
  * **Patrick**: wanted to run this by the group to make sure everyone is ok with this
  * **Graydon**: totally in favor of this entire process; you're not gonna want the GC flag to be turned on on trunk; we'll be bringing more machines into the rustbot cluster; one of the easiest ways to approach this is to have a secondary bot that builds with GC, if it fails doesn't block everyone else; should be simultaneously testing with optimizations on and off anyway
  * **Graydon**: hm, though turning off optimizations will actually improve our build times. weird.
  * **Patrick**: actually, the fast instruction selector (which we'll start with) runs very quickly, although the IR optimizations are way slower
  * **Patrick**: IR optimizations should be pretty easy to bring up, since they tend not to destroy identity of pointers
  * **Graydon**: excellent, if you need extra hands grab me
  * **Dave**: Eric Holk starting next week and may be interested in helping
  * **Patrick**: also, Brian got boxes working in addrspace 1; that was an annoyance that's now fixed
  * **Graydon**: I don't know what that means
  * **Patrick**: extra constant value you can put on pointers; LLVM IR has support for different kinds of pointers
  * **Graydon**: is it just trans work? or..?
  * **Patrick**: just trans
  * **Graydon**: just question of having all allocations of boxes get tagged properly
  * **Patrick**: a little tricky b/c LLVM treats them as incompatible types
  * **Patrick**: there's a bug on auditing every cast in trans and trying to remove; source of many bugs, and shouldn't need it anymore now that we're monomorphizing
  * **Niko**: so we hope

## Globals

  * **Graydon**: we don't have a good way of talking about globals, don't have them at all
  * **Graydon**: singleton tasks by string names -- not a good approach
  * **Graydon**: trying to figure out a way to do globals safely; bugs are on file, would love comments (just search for "global" and you should be able to find them)
  * **Graydon**: my current thinking: globals are a declarable thing you can declare safely, but access to them is only in unsafe code; libraries that unsafely use globals could have a new kind "const" or "static" that means deeply immutable; could do a broadcast to do an atomic update; both scenarios of single write single read / single write multi read (did I get that write?) are potentially useful
  * **Graydon**: could be very important for servo
  * **Graydon**: ROC had a design with versioned pages and versioned pointers; seemed pretty complex
  * **Graydon**: people working on Servo keep coming up with a single writer multiple reader scenario
  * **Patrick**: we actually implemented this already!
  * **Niko**: but with unsafe pointers
  * **Patrick**: yeah, it's unsafe everywhere
  * **Patrick**: regarding globals: I agree it would be nice to have some sort of isolated task construct; you'd like to be able to say "this task I'm spawning has a fresh set of globals"
  * **Brian**: don't you just not use globals?
  * **Graydon**: hard to enforce esp. if there's unsafe stuff floating around
  * **Graydon**: not sure that's anything other than just OS processes
  * **Patrick**: could also do a PLT scheme like you suggested at one point -- indirect into a table of globals
  * **Patrick**: other thing I'm kind of a fan of is different levels of unsafe: unsafe memory for full unsafety; unsafe globals that lets you write to globals but doesn't let you use pointers (similar to different kinds of casts in C++)
  * **Graydon**: might suggest you try phrasing that as different types of lint that you can set to error mode
  * **Patrick**: requires some sort of way to annotate what kind of unsafe things you're doing
  * **Graydon**: either annotating, or a way of checking
  * **Graydon**: we do have an attribute for controlling warning/error levels; the set of safe/unsafe patterns is kind of unbounded; then you start imagining a string argument, and then that's more like what our attribute system allows
  * **Patrick**: I'm fine with an attribute too
  * **Niko**: could we move unsafe itself to an attribute?
  * **Patrick**: attributes are optional, but unsafe breaks language invariants
  * **Niko**: I think we should use the term immutable instead of const; const has another meaning
  * **Graydon**: essentially, should we overload const or make a bunch of words that sound similar
  * **Niko**: const we only use for mutability
  * **Graydon**: we also use it for items in read-only memory segment
  * **Niko**: I just think it's a subtle distinction between shallow and deep immutability
  * **Graydon**: yeah, and that's distinguished by position
  * **Niko**: I just think that's confusing
  * **Niko**: I would be inclined to keep const the way you've defined it Graydon, but change the modifier that tells you if an lvalue is read-only
  * **Dave**: I like "readonly" or "ro"
  * **Patrick**: I like "ro"
  * **Graydon**: plausibly

## Documentation

  * **Graydon**: If anyone's in a documenty mood, would be nice to put together a guide-book for the compiler itself, and how the phases work, a little more than just the comments
  * **Graydon**: design-level invariants, strategies we want to maintain, that we don't talk about much
  * **Graydon**: a lot of state in our heads that's not stated explicitly and hard for newcomers to infer
  * **Graydon**: gcc has a hacking guide
  * **Tim**: maybe it would be a bit less intimidating to think of it as a collection of wiki pages rather than a guide book? ghc has a guide to compiler architecture in wiki form; might be an easier way to get started
  * **Graydon**: sure, could do wiki; I don't enjoy editing pages in the wiki, but you can use an editor since the wiki is a git repo as well

## Versioning

  * **Graydon**: we've all run into cases where we have to run make clean to get compiler working -- that's a terrible sign we have horrible bugs in the build system, linkage rules, interaction between metadata / linkage / inlining
  * **Graydon**: in process of adding some new features we've stretched definitions of versioning and compatibility to point where I'm not sure if any of us know when two crates are actually compatible with each other anymore
  * **Graydon**: we should aim to reconstruct a definition that is meaningful
  * **Graydon**: possibly an intern project? possibly one of us should start writing wiki notes; which compiler invariants are supposed to hold
  * **Graydon**: really hard to know when you have or don't have versioning working properly
  * **Graydon**: there's some specific stuff about simple versioning, but not sure now's the time to go into it
  * **Marijn**: my first inclination: do what macro-heavy languages do, and you have to recompile if you recompile anything you depend on
  * **Patrick**: I wouldn't throw out that much; as I understand it, most of our problems are due to stale versions of libraries sticking around and not using the metadata; we don't import the full version number
  * **Brian**: that's one possible thing
  * **Marijn**: our build system is already operating under the assumption it has to build everything in dependency chain; not running into this problem with changed inlines because we're not relying on those things
  * **Graydon**: definitely true
  * **Graydon**: can't resolve this all here; interesting question whether inline definition should be considered a different kind of dependency
  * **Graydon**: we've gone out of our way not to create a linkage error with a symbol unless its type changes
  * **Graydon**: if you've built one version with same name, same type, different body, not considered a linkage error; capacity to upgrade a library without rebuilding is in many way the purpose of having independent libraries
  * **Graydon**: currently our hashes don't hash the contents, just the type
  * **Niko**: if we keep version hashes, maybe the AST of inline functions ought to be part of it; you'll be linking against most recent version but you've inlined older versions
  * **Dave**: yeah, "inline" would then be a semantics-changing thing; seems like the kind of thing that would drive developers batty
  * **Graydon**: you can easily have bugs where you upgrade one thing but not another where their implementations need to stay in sync
  * **Niko**: not sure that's entirely true; your example is true, but... if the version numbers changed
  * **Graydon**: this is why we integrate the library version number. if user knows inconsistency is required, user can force a change
  * **Graydon**: users often want to issue updates w/o forcing recompilations. I would *prefer* just having a super-conservative rebuild and super-fast compiler, but users just totally balk at this.
  * **Niko**: maybe you changed the bug in an inlined function but allow people not to recompile
  * **Brian**: but what if you want to force a recompile?
  * **Graydon**: change the version number
  * **Niko**: do we have semantic version constraints?
  * **Graydon**: our current versioning system always says "give me the latest"
  * **Niko**: can I request specific version?
  * **Graydon**: if you ask for specific version you get only that version; otherwise you get whatever we find
  * **Tim**: is it possible to live in a world where we say "if I only import f and f gets inlined, if that changes I'll recompile, otherwise I won't recompile"
  * **Graydon**: that's the current model

## Language versioning

  * **Graydon**: other issue: language and metadata versioning
  * **Graydon**: awful issue. most difficult is grammar. an old compiler will bump into things that it can't even parse
  * **Graydon**: want some way to reliably to make syntax changes not come up as syntax errors
  * **Dave**: constraints: 1) keep the tax low (don't want a language version on every file!), and 2) how many backwards versions of the language does your compiler need to support?
  * **Graydon**: fear not, I have thoughts!
  * **Graydon**: how far back: don't think it's a requirement that we support lots of versions; essential that compiler can tell when it does and doesn't support something; there's a path for detecting and handling the error
  * **Graydon**: not even saying we should support old major versions of the language
  * **Graydon**: just want to be able to detect it
  * **Graydon**: tax has to be reduced, and has to be stable across all versions of the languages
  * **Graydon**: curiously enough, this dovetailed with a recent request for shebang
  * **Graydon**: because our model is based on single entry point, only has to go on one line -- top of crate
  * **Graydon**: second question is, how do you do those strings?
  * **Graydon**: our manual has a formal grammar which is wrong and unchecked; a formal grammar has a nice benefit that it drifts less commonly; formal grammar should change less commonly than the code in the parser
  * **Graydon**: if we versioned the grammar by hashing the parser it'd change every day, but if we hash a formal grammar that shouldn't change nearly as often
  * **Dave**: why the has?
  * **Brian**: could just use the hash internally to know we need to bump the language version
  * **Patrick**: could make the shebang optional
  * **Graydon**: oh yeah, totally optional
  * **Graydon**: so, there's the same issue with metadata reader
  * **Tim**: is the metadata grammar specified anywhere?
  * **Graydon**: that would be step 1 :)
  * **Patrick**: can write a DTD for EBML

## Administrative

  * **Graydon**: we have some new machines
  * **Graydon**: I'll probably have them not all do the same thing
  * **Graydon**: if anyone has thoughts about what they'd like them to do, let me know
