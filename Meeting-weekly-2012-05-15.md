## Attending

Graydon, Tim, Patrick, Paul, Niko, Eric, Lindsey, Dave, Brian

## Resolve

  * **Niko**: resolve problems, to do with exporting / items not being found
  * **Patrick**: for me it usually has to do with exporting and then importing impls, import * works around
  * **Niko**: assume part of the problem is reachability computation, but I've tried to fix it a few times and it hasn't solved it
  * **Graydon**: my main concern is that there's a pile of code that's not merged, and will bitrot
  * **Tim**: timeline?
  * **Graydon**: asap
  * **Paul**: I could look at it
  * **Niko**: hesitant to put an intern on it, b/c it could take a while
  * **Graydon**: current state: there's a rewritten one, but it doesn't get all the way through the current standard libraries; I believe it's "done-ish" but not debugged
  * **Dave**: suggest it's best for a full-timer
  * **Tim**: I can do if it's higher priority than incremental compilation and classes
  * **Niko**: definitely higher than incr comp
  * **Tim**: remaining classes bugs are pretty small; someone could jump on them in a day
  * **Patrick**: can we start using classes?
  * **Tim**: I'd like to finish dtors
  * **Patrick**: we can use them without destructors

## Dvecs

  * **Niko**: I did some work on dynmically-growing vector type; wuld like to push it; compilation is a little slower; vector in isolation is as fast or faster than existing code, though I don't know why
  * **Niko**: it's like a 3% perf hit, and can probably be optimized over time
  * **Graydon**: assuming you're not replacing all uses of vector growthin our compiler with it?
  * **Niko**: not all, but a couple anyway; big one is small int map -> dvec gets slower; seems like the layering of abstractions
  * **Graydon**: you sure the compiler's getting slower vs having more to compile?
  * **Niko**: pretty sure
  * **Patrick**: so it's the bounds checks and stuff like that?
  * **Niko**: couple places, maybe; does other checks, tries to grow vector in case it gets too big; these things pile up
  * **Niko**: could be solved with some inlining, or...
  * **Patrick**: we can probably optimize those
  * **Niko**: I wanted to warn people that this will happen, since it does cause a measurable performance decrease
  * **Patrick**: let's file a bug to optimize small int map
  * **Niko**: want to do some work towards finishing boxed vectors; over time as our vec story gets straightened out we'll optimize
  * **Graydon**: kinda curious where the code for this is
  * **Niko**: my repo under dvec branch
  * **Niko**: background: working on analysis to ensure unique pointer accesses are safe, but our vectors are currently unique; the dvec class replaces that
  * **Niko**: another thing I wanted to bring up, kinda an RFC topic: choose between copies vs detecting and making copies
  * **Graydon**: I think detecting and failing and a bug to fix in the future is good; don't break the invariants of the map
  * **Graydon**: long-lived iterators "leasing" == bad
  * **Niko**: that's what Java does

## Macros

  * **Paul**: lot to talk about; should be one piece I can take out and work on: parser changes necessary to allow macros to take paren-balanced pile of lexemes
  * **Niko**: not sure everyone knows what we've been talking about
  * **Paul**: allow macros to take open paren and close paren, between them anything so long as it's balanced, just a pile of lexemes that need balanced pairs of delimiters
  * **Patrick**: like camlp4, right?
  * **Graydon**: no, camlp4 works on basis of characters; scans till it finds close bracket
  * **Graydon**: where I initially left the design of the rust macros system: two separate delimiter forms: round brackets enclose well-formed expressions; curly braces enclose curly-balanced character strings; if you wanted bit-patterns, regexps, xml, etc you could put them in braces
  * **Graydon**: didn't leave any space in that design for lexeme lists a la what Paul is talking about
  * **Paul**: that's the glorious grand plan
  * **Patrick**: it is?
  * **Niko**: uhh..
  * **Niko**: is this just for MBE macros? or for everything?
  * **Paul**: I think a reasonable space is one macro invocation syntax that takes AST's, and one that takes sequences of lexemes
  * **Niko**: what does having lexemes over characters that are paren-bound give you?
  * **Paul**: when dealing with characters, going to have to implement a lexer; having different lexer rules is going to drive your syntax highlighter crazy
  * **Niko**: if you want your own lexer you'd have to implement your own
  * **Paul**: I don't like having the option of having a different lexer
  * **Patrick**: lisp has reader macros
  * **Paul**: if you want your thing to be string-aware, you won't be able to write this:
```
<<[ "]>>"
```
without having to write a special string-lexer
  * **Paul**: and you want to be able to syntax-highlight etc and have some measure of consistency inside these things
  * **Niko**: kind of violates the macro abstraction in a way
  * **Paul**: I don't feel like it's worth the benefit, unless there's a really strong need
  * **Patrick**: well, regexps are important
  * **Dave**: you could build them in
  * **Patrick**: or special quote
  * **Graydon**: I recognize what you're saying; it's all valid, but then you have to put regexps inside of strings, and that aesthetically upsets users
  * **Dave**: I think I agree with Graydon that you'll want custom quoting for regexps
  * **Patrick**: the current AST-based mode could be replaced with the lexeme-based mode; the current one doesn't know what non-terminal to start with, which is the main problem if you do a lexeme-based mode; the macro gets to choose the interpretation
  * **Graydon**: I agree with that, like the sound of it, w/ one exception: are you talking about S-expressionizing everything? you wanna do nesting; do you want nested lists inside of nested lists?
  * **Dave**: I would like to see an RFC; we'll do more details there

## Reflection API

  * **Graydon**: type descriptors, callbacks to walk structure of type descriptors
  * **Graydon**: just so you know this is going on in the background
  * **Graydon**: does seem to be working

## Stability

  * **Graydon**: wondering how long people feel before we should be stabilizing?
B: 7 months.
  * **Graydon**: ok!
  * **Patrick**: I just don't want to bake in any decision we made for expedience; lot of stuff that currently exists only b/c it was the easiest at the time
  * **Graydon**: examples?
B: I made a design mistake I won't tolerate: companion modules -- implicitly search filesystem for files, it was a big mistake
  * **Niko**: is it a mistake to have companion mistakes at all, or the places we look for them?
B: mostly the places we look for them, I think
  * **Patrick**: argument modes and our 5 different argument modes
  * **Graydon**: yeah, they've gotta go
  * **Niko**: maybe break out features?
  * **Dave**: yes, and we can figure out which features are post-1.0
  * **Graydon**: yeah, dependency tracking would be good. not to get all MS Project...

## Crates

  * **Graydon**: issue 2369 ( https://github.com/mozilla/rust/issues/2369 )
  * **Graydon**: the crate abstraction is from a long time ago. based on assumptions that we weren't gonna monomorphize, never were going to do CCI, were never going to serialize/deserialize AST's... none of those are true!
  * **Graydon**: the abstraction is nowhere near as pithy in my mind
  * **Graydon**: principled overhaul of compilation model? possibly around managing the form of inter-module linkage independently of the versioning of them
  * **Graydon**: incremental compilation is a versioning problem
  * **Graydon**: so, I'm wondering if the whole crate abstraction is one of those "designs made for expediency" we should get rid of
  * **Tim**: I think the answer is probably "yes" that they should be significantly remodeled
  * **Patrick**: .net has the idea of assemblies; not without precedent
  * **Dave**: I think two-level is natural, but I'm ignorant of the detailed design issues
  * **Patrick**: what I worry about is Servo, where we have half a dozen dependent crates already; what happens (and I think it's very common, even in web apps) in development, is that you depend on all these other libraries; you create your crate and you want to erase those other dependencies; don't want others to have to see that; people do this with ruby rbenv and python; they take it to the extreme b/c it wasn't designed up front
  * **Dave**: I think it's tied in to the kind of deployment (executable versus directory structure web server vs...)
  * **Graydon**: yeah, I think for servo it makes sense to have a single statically linked executable
  * **Graydon**: what I want is to file the tool down so that the distinctions are as clear and simple as possible
  * **Graydon**: maybe worth an artifact in the system that's symbolic of project names and acyclicality; git is similar with submodules
  * **Graydon**: maybe what we call a crate is minimum necessary line drawn between two directories
  * **Patrick**: that makes a lot of sense to me
  * **Niko**: that sounds really good. I definitely like the UI concept of "crate" but the details
  * **Dave**: I really feel the two levels are a natural part of the landscape, particularly with acyclic dependencies at library level, cyclic dependencies at module level
  * **Graydon**: acyclicality goes hand-in-hand with separate versioning
