## Attending

dherman, brson, nmatsakis, tjc, graydon, pcwalton

## Status updating

  * **Graydon**: can we do weekly status update emails?
  * **Brian**: can we do them in blog format?
  * **Niko**: +1

## rustpkg as package manager (not "cargo")

  * **Graydon**: rename cargo to rustpkg. anyone mind?

## build / pkg / maint strategy

  * **Graydon**: talked with erickt last week. he wants to work on the build infrastructure. hashed out a strategy; wanted to reproduce that here
  * **Graydon**: multilayered strategy
   * lowest layer: replica of build tool erick wrote long time ago for Felix language, called fbuild. simple build mgmt system: just cached function evaluation. write a build script, imperative seq of commands. commands are defined as function calls that may not evaluate; may just fetch cached result. the memoize via filesystem
  * **Niko**: like ccache
  * **Graydon**: yeah, or dan berstein's build system is similar. uses language's evaluation process. needs some metadata on what a function argument is
  * **Graydon**: based on tasks so it can be parallelized
  * **Niko**: to parallelize you have a central director task? like a future-y approach?
  * **Graydon**: yeah, any of memoization tasks are doable as subtask
  * **Graydon**: that's quite built-task-agnostic; doesn't specify build logic, just an evaluation library
   * next layer: rules for operating clang, filesystem targets, normal build-y activities. probably part of libpkg or librustc will use the build driver to issue commands. libbuild becomes a switching point for issuing commands to other tools, such as compiler driver, downloading stuff, filesystem, running git, etc
   * on top of that: two tools:
1. rustpkg: normal fetching, downloading, running builds, as much as possible based on attributes of project. try to make it so that it works by analysis of crate file. one benefit: very good at dynamic builds: builds that construct rules on the fly. would like to not have users providing build scripts except as last result. it's an escape hatch.
2. rustmaint: library for building rustc itself using this technology. probably not declarative, just a custom build, because it involves weird things like bootstrapping.
  * **Graydon**: so this is the broad high-level approach we were thinking of
  * **Brian**: sounds awesome
  * **Patrick**: sounds good to me
  * **Niko**: question: framework for dispatching and memoizing tasks: can that be made pretty general?
  * **Graydon**: very very general
  * **Niko**: applicable beyond build systems. cool to have as a fundamental library
  * **Graydon**: could be used for driving test suite, things that happen inside compiler
  * **Niko**: I would imagine could be refactored to serve as one of several parallel primitives we make available. cool, I like it
  * **Graydon**: Erick's written this before so he'll be helpful for this, can draw on his expertise

## Graydon updates

### license

  * **Graydon**: we're very close. dual MIT/Apache2 looks promising. most everyone seems happy with this.

### buildbots

  * **Graydon**: able to be run on AWS
  * **Graydon**: put it on public internet last week: buildbot.rust-lang.org is up and hasn't fallen over so far
  * **Graydon**: eventually should start making that the canonical address

### quasiquote

  * **Brian**: new one replaces old one?
  * **Graydon**: yes, I want to. previous one works on string spans. fundamentally disagrees with token trees as concept.
  * **Niko**: problem with old one was not being able to substitute in some places I wanted to. addressing those limitations?
  * **Graydon**: yep. should be able to substitute anywhere you can in a MBE-style macro.

## deriving

  * **Graydon**: is there a version of this that threads a value through all deriving calls?
  * **Patrick**: hasn't been much design work on this. mostly I'm doing the trans work that will be broadly applicable to whatever design.
  * **Patrick**: in order for this to be useful definitely needs to take values and thread them through
  * **Patrick**: was thinking all parameters get threaded through as is, except the boolean "other" parameter
  * **Niko**: should probably try to hammer out design of some sort. I have some use cases that are not boolean. doesn't have to block you
  * **Patrick**: am wary of making design too general; leads to too many minor incremental "generics" improvements; would rather fall back to macros. erickt already was asking for field names
  * **Niko**: what does that mean?
  * **Patrick**: passing field name as a struct
  * **Niko**: yeah that's too much
  * **Niko**: but I would like to gradually add in new modes of deriving
  * **Patrick**: I would hope that we don't get there. want deriving to be simple. it's a convenience
  * **Niko**: not sure I agree, because it's there as a convenience
  * **Niko**: I want it to be a compiler-provided macro. a code generator, a boilerplate eliminator
  * **Patrick**: yeah but that's really hard. endless stream of haskell papers on this
  * **Niko**: I think we agree; don't wanna do research, just want conveniences
  * **Dave**: not sure the "too many research papers" is a fair argument
  * **Patrick**: there are lots of papers on "we added this feature to GHC to support SYB"
  * **Niko**: I just want to support some good use cases
  * **Patrick**: don't wanna keep extending this over time
  * **Dave**: why promise not to add functionality? strange promise to make
  * **Niko**: why not just respond to "this boilerplate is common, let's add support"
  * **Patrick**: it's metaprogramming in an ad hoc way; macros are general purpose
  * **Dave**: he's saying let's not keep adding special cases to compiler
  * **Niko**: then don't have deriving, just use macros
  * **Patrick**: macros don't have enough understanding of types
  * **Niko**: and that's why I think we should have a more expressive deriving system
  * **Patrick**: we also have reflection. if not perf-critical, can go through reflection
  * **Niko**: not sure we're that much in disagreement
  * **Dave**: this sounds like post-1.0 to me; every 1.0 language has some boilerplate in it
  * **Graydon**: I'm curious: is this or isn't it being done as a macro?
  * **Patrick**: right now open-coded in trans
  * **Graydon**: *could* be done as syntax extension?
  * **Patrick**: no, has to run in middle of typecheck, because of coherence tables
  * **Niko**: I'm not sure this is true. I'm not sure it has to be designed that way. ast-serialize is a deriving technique & it works
  * **Patrick**: yes but goes on declaration of type
  * **Niko**: right now: `impl t : T` -- if we change to annotation on type, with `derive T` could make it work
  * **Graydon**: syntax extensions run before resolve
  * **Niko**: so does ast-serialize
  * **Graydon**: can't name stuff and expect it to be able to find what it points to
  * **Niko**: can just reproduce the name; that's what ast-serialize does
  * **Niko**: would be nice if we didn't have to modify trans for this
  * **Brian**: that would be nice, to not have this in trans
  * **Patrick**: that's true
  * **Niko**: not completely convinced it doesn't work
  * **Patrick**: it might work
  * **Graydon**: sympathetic to general problem, which is hard to get past: have to pick phases for general points of extension. have this thing in the front-end: sometimes what configuration to run before expansion. want to config off a macro. even in that narrow phase ordering occasionally want things in another order.
  * **Niko**: may be able to solve this with the ordering we have now
  * **Patrick**: still not sure it works, have to be able to introspect on trait you're deriving
  * **Graydon**: maybe worth pushing that idea. possibly introducing some extension point post-type-checker
  * **Dave**: maybe something John Clements could look into when he's here
  * **Niko**: easier for one-thing trait like Eq, but might be harder for things with more involved traits. might need some post-resolve extension point
  * **Patrick**: the code in trans is not really that bad
  * **Graydon**: it's not the badness, just the special-purposeness that I was objecting to
  * **Niko**: in general compiler is oriented to every syntactic sugar is ending up in trans, not hiding any of it
  * **Graydon**: that was my doing :) starting to move away from that
  * **Niko**: we should have some other expansion steps before trans
  * **Patrick**: I disagree with that. I think it's fine the way it is
  * **Graydon**: big enough topic that it requires a mailing list post

## deriving syntax

  * **Brian**: right now, just magic when compiler sees right pattern. would be nice to have real syntax for it
  * **Niko**: don't even know... `impl foo;` ?
  * **Patrick**: yeah
  * **Brian**: so if you do braces it doesn't derive?
  * **Patrick**: yeah, that should be fixed. what I was thinking: `impl foo : bar;` same as `impl foo : bar { }` -- if you leave method off, tries to auto-derive it
  * **Brian**: not that crazy about it
  * **Graydon**: yeah, I think I prefer more explicit
  * **Dave**: just trying to conserve keywords? could be contextual
  * **Patrick**: no contextual, we already agreed. trying to conserve keystrokes
  * **Niko**: I think trait should be annotated, but I agree with Patrick on conserving keystrokes
  * **Brian**: seems similar to default methods. if we could frame it as default methods that might be more consistent. just marking trait would be fine with me
  * **Niko**: yeah, I like that
  * **Patrick**: mark method?
  * **Brian**: that would seem to fit with default methods better
  * **Niko**: yeah
  * **Patrick**: yeah
  * **Brian**: is method derivation always independent of trait derivation?
  * **Niko**: depends how ambitious we get. I guess for now it's per-method
  * **Brian**: seems like a reasonable assumption
  * **Patrick**: dunno what it does with multiple methods
  * **Niko**: I like the idea you can tag the method
  * **Brian**: I'm satisfied
  * **Niko**: me too

## unsafe supertype

  * **Niko**: idea: *T >: &T
  * **Niko**: can always convert region pointers to unsafe pointers
  * **Niko**: one implication: committing to idea that &T is always a raw C pointer, never a root to the GC. I think that's good, though.
  * **Patrick**: would be very surprised if we haven't committed to that in all sorts of other ways
  * **Graydon**: I think we have
  * **Graydon**: reminds me of how we talked before about treating * as a special region
  * **Niko**: if you have a region-parameteric function, wouldn't apply to *T. treating as a region is not sound, I think. at least I don't know how to make it sound
  * **Graydon**: don't remember why it was or wasn't sound
  * **Niko**: just doesn't behave like other regions because it's unsafe
  * **Patrick**: has to be in an unsafe block, which is different from region-parameteric functions
  * **Niko**: might be able to concoct some rules but I remember running into weird issues
  * **Niko**: another thing that makes it different from other regions: can convert from any pointer of any lifetime to *T. I guess that makes it like static
  * **Graydon**: I thought it was a super-region
  * **Niko**: maybe you could if it were a super-region of static. but then would need rules for `unsafe` keyword. apply region-parameteric function, must be in unsafe block.... just seems complicated
  * **Patrick**: seems wrong to me
  * **Patrick**: anyway, practical upshot is no longer needing the to_unsafe_ptr coercions
  * **Niko**: when we tried to switch to ^T it was already unappetizing
  * **Graydon**: not talking about way you write it, just what it's a synonym for
  * **Patrick**: I think an unsafe region interacts badly with region-parametric functions
  * **Niko**: I'd rather keep it separate
  * **Graydon**: this makes it a subtype?
  * **Patrick**: no, supertype. just not a region
  * **Patrick**: supertype of all region pointers
  * **Graydon**: nomenclature-wise: I'm trying to refer to them as raw pointers rather than unsafe, because they may be safe
  * **Dave**: I think unsafe means "Rust can't prove it's safe"
  * **Graydon**: oh I know why: just for renaming modules to avoid colliding with keywords

## extern functions

  * **Brian**: we've had design for new extern fn types. mostly I understand how we generate wrappers on demand when we call those functions.
  * **Brian**: sometimes we call those functions on Rust stack, require stack switch. we also have stuff where we don't do the stack switch
  * **Brian**: I don't understand how this new system decides which
  * **Niko**: that could be a separate ABI, like "c-rust-stack"
  * **Niko**: OTOH, it's an orthogonal attribute so kind of lame
  * **Niko**: has to go in type somewhere
  * **Brian**: to make this work seems like it has to go in type
  * **Niko**: or call site, but would have to make syntax for that
  * **Brian**: I guess making it part of ABI is okay. will have to have a convention
  * **Niko**: or else ABI could be: name followed by optional semicolon with other information
  * **Brian**: could be an arbitrary attribute
  * **Graydon**: maybe just a sequence of identifiers, comma-separated
  * **Patrick**: was gonna be `extern "C"` -- `extern "C, rust-stack"` ? pretty ugly
  * **Niko**: not too concerned about aesthetics
  * **Graydon**: see if you can come up with anything that isn't ugly. just kind of an ugly thing
  * **Patrick**: how about `extern "C-noswitch"`
  * **Niko**: but you need every ABI "-noswitch" version
  * **Patrick**: that's okay
  * **Brian**: going other way: when cast Rust function to noswitch; little wrapper that adapts calling convention, with no stack switch
  * **Graydon**: yeah, 2x2 matrix of wrapper obligations
  * **Niko**: didn't we have all these ideas to get rid of stack switching?
  * **Brian**: seems like in 64-bit stack switching may not be best strategy. might want to prepare for future with no stack switching at all
  * **Patrick**: these days I run all Rust code with min-stack=BIGNUMBER
  * **Brian**: different issue
  * **Patrick**: no, same issue. stack switching has cost
  * **Brian**: min-stack doesn't save you switching, just growing
  * **Patrick**: oh. saves you growing. but related
  * **Brian**: similar
  * **Graydon**: why switching bad on 64-bit?
  * **Brian**: not bad, but mmaping of stacks more performant on 64-bit
  * **Graydon**: certainly performs better...
  * **Patrick**: as I understand things, reason for segmented stacks is to avoid running out of space
  * **Graydon**: avoids committing to space you're not gonna use
  * **Niko**: most systems grow stack gradually anyway
  * **Graydon**: we are going to be with 32-bit for a while, at least on phones
  * **Patrick**: even on desktop
  * **Graydon**: I agree it might be nice to have a version that doesn't waste cycles if it doesn't need to
  * **Niko**: a shame this leaks into ABI specifications. guess it could become a noop

## library design discussions

  * **Dave**: do we want a longer meeting to discuss library design? maybe post-0.5
  * **Graydon**: plates are full, but I agree we should have that discussion
  * **Niko**: some pieces not yet in place, but will be good
  * **Graydon**: a work week would be good
  * **Dave**: possibly a work week would be hard to coordinate; vidyo can serve as a fallback
