# Attending

Patrick, Niko, Sully, PaulS, Graydon, Jesse, Elliott, Ben, Brian, Tim, the ghosts of departed Lindsey and Eric

# Status updates

- all of bblum's projects are done and in (ARCs) (bblum)
- bblum and sully and pauls leave this week

# One-shot closures

  * **Patrick**: we need them: 1) generally accepted in affine type systems that you want affine closures if you have affine types
  * **Patrick**: reason has become painfully apparent to interns as well as to myself in Servo; using cells way to much
  * **Patrick**: basic problem: you can't move out of a closure
  * **Patrick**: liveness checker assumes all closures can be called more than once; prevents nice abstractions; specific case: time function takes block and times it; adding that made it impossible to move variables inside that block
  * **Patrick**: Graydon's proposed syntax was once fn -- I think that's perfectly fine; subtyping relationship probably
  * **Niko**: no, I hope not
  * **Niko**: Re: once fn, once plays role of sigils like @; you're basically only gonna pass these by value
  * **Patrick**: you sure?
  * **Niko**: yeah
  * **Patrick**: what about spawn?
  * **Niko**: what about it?
  * **Patrick**: you pass unique closure to spawn
  * **Brian**: send says what's in it; once says how you can use it
  * **Niko**: that's true. we need to talk about it, I guess
  * **Niko**: basically two use cases: 1) access things by ref (like time example) -- stack closures, and 2) spawning case
  * **Ben**: there are other cases, but about the same: task unkillable
  * **Niko**: sounds like stack closure to me
  * **Ben**: yeah
  * **Niko**: one thing weird in current closure stuff: type doesn't tell you whether it's accessing by ref or not
  * **Patrick**: little confused. what's diff at runtime between & and ~ fn?
  * **Niko**: where env is allocated
  * **Patrick**: does that really matter? I guess it does when you need to free it
  * **Niko**: we should talk offline. but I agree it's important
  * **Brian**: I think it's only a little important. we have workarounds; it makes things more complicated still.
  * **Ben**: I think it'll make code drastically simpler. fine writing option dance, but I don't really wanna sell a language that requires the dance everywhere
  * **Patrick**: when you hit it, it's extremely painful
  * **Brian**: I'm familiar! But we should be cautious about complexity
  * **Patrick**: it's unheard-of in langs with affine types
  * **Dave**: langs with affine types are virtually unheard of!
  * **Graydon**: gonna sound like a broken record, but I continuously do not understand the motive. why do you need to move out of a closure?
  * **Patrick**: you need to move generally for a lot of reasons: sending value to another task
  * **Ben**: sending a pipe-end or an arc
  * **Patrick**: passing a string around by move, for example
  * **Patrick**: any time the thing you want to move is an upvar, and you happen to be inside a stack closure, and the thing you're in is an upvar
  * **Patrick**: happens a lot b/c we tend to use stack closures for block-like things
  * **Graydon**: maybe Niko's differentiation is relevant to understand this
  * **Graydon**: whether closure owns it or not seems like a big difference
  * **Patrick**: if closure created it, it can move
  * **Niko**: in no case can we do it if it's an upvar. that's the salient issue
  * **Graydon**: so entire point of topic is to move upvars
  * **Niko**: yes
  * **Graydon**: and other case is you're sending on a channel and want to continue doing move-type stuff on other side of the send
  * **Brian**: I find it happens more often with unique closures
  * **Patrick**: why usually want to move out of unique closures? like pipe ends?
  * **Brian**: yeah, various unique things I need to shuffle around. task has one particularly egregious instance of option dancing. moving pipe end through many many closures, and it's horrible
  * **Graydon**: enforced statically in sense that compiler will consume closure value when it's called?
  * **Patrick**: just prevents you from calling it multiple times. don't know if consumes it
  * **Graydon**: think it must
  * **Niko**: yeah, think so
  * **Niko**: whether caller or callee does construction... probably callee, b/c it knows more -- it knows if it's a stack closure and no destruction necessary
  * **Graydon**: callee can pass to one sub-callee or call once itself
  * **Graydon**: how do you statically enforce this?
  * **Niko**: same as any other non-copyable type
  * **Graydon**: but we want to pass references further inwards. current frame can either call or pass it further inwards
  * **Ben**: if passes by value, great, but by reference, can't do anything else
  * **Sully**: can't call it by ref, b/c can't deinitialize
  * **Niko**: like any other non-copyable
  * **Graydon**: but it's not copying. you're trying to restrict number of times people call it
  * **Niko**: here's why I'm saying same thing, in a way: when you call it, you consume the thing you called. just like moving out of something
  * **Graydon**: but one of the use cases is explicitly not owning (as in owning memory) model
  * **Niko**: closure doesn't own it, but function given closure owns a reference
  * **Sully**: but they don't own the closure
  * **Niko**: that's what I'm saying, we can't do it w/ a region pointer, b/c they wouldn't own it
  * **Sully**: need special once fn that still works like an &fn
  * **Niko**: no, not referenced by borrowed pointer. in general a type can say: I contain region ptrs but I am not myself a region ptr. exactly like record
  * **Brian**: still doesn't specify where it's allocated
  * **Niko**: I'd like to avoid specifying that in the type, b/c doesn't matter to caller
  * **Niko**: it kinda does b/c of freeing matter, but we can finesse that
  * **Niko**: what I had in mind: if you call task::spawn, it says "I need a once fn with region bound of static," so can't capture anything on stack, only static data
  * **Niko**: region bound distinct from static. it's very complex, is the truth. my thought was if you need a static one, we'll allocate one on the exchange heap. sort of annoying b/c it's taking clues from how you're saying you're going to use it. if you give us restrictions, we'll allocate it cheaper. but, ... yeah.
  * **Niko**: does that make sense?
  * **Brian**: a little
  * **Niko**: you'd say "give me a once-fn I'm only gonna use it in this fn" by giving a region bound (maybe once &fn or something) and that's saying it can't escape. then caller can put it on stack b/c it won't outlive stack frame
  * **Patrick**: I think we need a concrete rfc before we can proceed
  * **Jesse**: unsafe::forget(...)?
  * **Graydon**: dunno how that's relevant since Jesse's sound is not working
  * **Jesse**: just being silly
  * **Patrick**: this ties into something deeper: to what degree separate region bound concept from sigil on front of closure type
  * **Patrick**: kind of like the way it is now, but once fn is really important IMO. we need a proposal
  * **Brian**: also needs to include fix for type bounds on closure; completely busted now
  * **Niko**: I have that fix ready to push
  * **Niko**: not sure what you mean by how it's busted
  * **Brian**: type bounds are based on old kind system; now we just have random traits with no hierarchy; completely unsound ATM
  * **Niko**: copying matter in particular?
  * **Brian**: yes
  * **Patrick**: yeah, you can copy non-copyable easily
  * **Sully**: you can do so in a lot of ways
  * **Niko**: we should fix that but that's separate
  * **Brian**: it's going to play into this syntactically
  * **Dave**: let's move on since needs RFC
  * **Dave**: need to stabilize it soon though
  * **Patrick**: we need to have a hard discussion about fn types
  * **Patrick**: anyone opposed on principle?
  * **Graydon**: not on principle, just concerned about complexity. and concerned I don't understand our function types anymore. worried that it doesn't hold together
  * **Patrick**: yeah, we should think about overhauling
  * **Graydon**: that's a scary word, we worked hard to get where we are
  * **Patrick**: minor overhaul
  * **Graydon**: may just be a documentation/explanation issue. same issue with traits/impl but I'm getting the hang of it now
  * **Niko**: suspect you understand better than you think; problem is we haven't decided what they should be
  * **Graydon**: fact that they're simultaneously describing storage of env and contents of env, and I used to think those two things were joined at the hip, but I don't think they are anymore
  * **Graydon**: anyway
  * **Graydon**: I'm curious about the concept of dynamically enforcing one-shot closures and whether that's a plausible compromise
  * **Niko**: good thing to keep in mind; effectively we already do that with cells and swapping
  * **Graydon**: wouldn't want one-shot to involve lots of boilerplate, but if it can be localized...

# Type/module namespace unification

  * **Patrick**: Niko had a proposal
  * **Patrick**: gist is: we haven't found a satisfactory way to say `new`. would be nice if we could say T::new, that's intuitive
  * **Patrick**: touch on idea that "traitless impl" -- bad name -- really just a group of functions related to this type, so happens to be callable
  * **Patrick**: kind of like a module in a way
  * **Patrick**: contrary to mail I sent on weekend, don't think we need to make it full-fledge module, Niko thought that's confusing, could just be bunch of functions
  * **Graydon**: would resist thinking of it as module since they play with imports/exports; already realized you had to phase that resolution process
  * **Patrick**: could in theory import from them too, but... if you re-export types maybe not
  * **Graydon**: you'll tangle resolver and type checker
  * **Dave**: don't play crazy phase games
  * **Patrick**: ok, not really modules. but could be callable with :: notation. static functions w/ traitless impls
  * **Niko**: static functions anywhere, right?
  * **Patrick**: yeah, could be callable using :: which is nice to disambiguate. solves our `new` problem pretty nicely. `T::new` is what people would like to write
  * **Graydon**: not bad. curious what if it goes the other way: wind up being able to use type names in module paths in context where we need a module path, like `use` declarations? can a `use` declaration mention a type? I don't think they currently can, we're very restrictive of what can be there
  * **Niko**: I was proposing path... today consists of some number of modules and tail item
  * **Patrick**: won't be a module in future
  * **Niko**: I was proposing some number of modules, optionally a type, then tail item
  * **Niko**: would be nice if you could import from them. separate question
  * **Patrick**: way it plays with re-export scares me
  * **Patrick**: if you can re-export modules and types, you lose the phase distinction, back in resolve crazy-town
  * **Graydon**: scary swamp we just managed to drag ourselves out of
  * **Niko**: not end of world if to call static thing you have to qualify with type
  * **Graydon**: I don't mind it
  * **Dave**: I'd give a try and see if it's painful before doing something complicated
  * **Patrick**: doesn't really make sense to import `new` and that's the main case
  * **Niko**: static functions Sully introduced can be in namespace, but they'd belong to their trait, and have to write T::f
  * **Patrick**: if you start forbidding re-exports you might make it work, but I'd have to think about it
  * **Graydon**: would suggest seeing if this can be done via something smaller than same namespace, like what Niko said, extending what's allowed in paths
  * **Niko**: if they're not same namespace, could have module or type and it's ambiguous
  * **Patrick**: was thinking of implementing by exempting primitive types
  * **Graydon**: can't have modules and types of same name, because they're at same level of item
  * **Niko**: you can b/c they're in diff namespaces
  * **Niko**: in practice, CamelCase types and lowercase module names won't conflict; putting in same namespace is not terribly imposing
  * **Graydon**: then lemme phrase opposite way; if you put in same namespace but continue to restrict resolution phase that considers modules
  * **Niko**: just b/c in same namespace doesn't mean treat same everywhere, just can't have two with same name
  * **Dave**: I feel like reducing number of namespaces is good for usability anyway
  * **Niko**: there's a sweet spot; I like meta and value, and that's what we're talking about
  * **Brian**: I'm not convinced about this yet; I like reducing namespaces, but problem this is solving only solves one specific problem; leaves several problems: can bury constructor in type, but for more complex objects there are other things associated with the type. actors for example: set of messages. pattern to do that is module containing the messages, prefix message with name of module. if we go to this system, constructors are solved but other things need to be named, associated closely with a type
  * **Niko**: good point
  * **Patrick**: could allow enums and other things inside traitless impls. don't see any incoherence there
  * **Brian**: we could, it's true.
  * **Niko**: I've often wanted that for other reasons
  * **Graydon**: all just about naming, right? don't want foo module with foo type?
  * **Niko**: not exactly. that would still probably happen
  * **Graydon**: currently I can put my constructors in module containing trait
  * **Niko**: I think what's unsatisfactory is importing that becomes fairly complicated. that's really the only problem
  * **Graydon**: I could put my constructors in a module with same name as type, next to it. when I import type name I import module with static functions. just feels redundant
  * **Patrick**: but that doesn't work b/c you have to say use mod. this is the phase distinction
  * **Patrick**: you need import module to be syntactically distinguished from anything else to make resolve coherent
  * **Graydon**: put another way, a module with a self type is really what we're talking about?
  * **Dave**: does that make sense?
  * **Niko**: essentially same thing; whether you say each type can have associated module, or both modules and types can contain items
  * **Patrick**: think it works if you say impl is a special module in which self has a meaning; meaning of self is determined by type after impl
  * **Patrick**: not an ordinary module; you can't import from it b/c it's tied to a type and types can be re-exported
  * **Patrick**: but it's otherwise a module in all senses
  * **Patrick**: basically what I was getting at in my email; it's basically a module. other thing is: self has a meaning & therefore can be used to attach methods
  * **Niko**: I think it makes sense but I don't know
  * **Patrick**: addresses Brian's issues
  * **Brian**: yeah
  * **Patrick**: seems delightfully MLish to me
(laughter)
  * **Graydon**: reminds me of, honestly, when we first talked a couple years ago about dot syntax and ad-hoc overloading. didn't use the word impl, talking about self-types. talked about a distinguished declaration inside a module called module's self-type declaration
  * **Graydon**: talked about `type self = ...` as associating that module with dot-calls
  * **Graydon**: difference is that now ability to make dot-calls is now tied into traits
  * **Patrick**: I think traits could well be separate; traitless impl is kind of just a totally separate thing from declaring its conformance to a trait; declaring conformance to trait does not anchor a name anywhere in hierarchy
  * **Graydon**: yeah, that's a resolution thing for later
  * **Patrick**: for me feels weird to have same keyword for both, but I can't think of a better keyword
  * **Brian**: might kind of prefer module self-type to introducing another kind of module; seems like fewer new things
  * **Niko**: same thing
  * **Dave**: syntactic difference that matters though
  * **Niko**: thing about self-type: how would you name this type inside module? self I guess
  * **Graydon**: self
  * **Niko**: outside the module you'd write the name of the module?
  * **Patrick**: confusing & hard to grep for
  * **Graydon**: can use module name as type inside the module as well
  * **Brian**: you don't always know the module name, if the module is a file
  * **Graydon**: in context where you don't know module name, use name self
  * **Patrick**: hard to grep for
  * **Dave**: maybe you don't want to tie it to file name, might want to rename
  * **Brian**: I don't like completeness of module to depend on what's going on outside file
  * **Patrick**: Java for example: say name of class, would be weird to leave it anonymous
  * **Brian**: also would screw up camel-case rules; types would be lowercase again
  * **Niko**: modules with self-types would by convention be uppercase
  * **Brian**: ugh
  * **Graydon**: sort of a wash then. my one thing is that modules with self-type is lower cognitive load, can eliminate impl keyword
  * **Patrick**: what about trait conformance?
  * **Graydon**: just put that in the module declaration (with a colon and list of traits)
  * **Niko**: when you implement a trait, that's a separate thing, but that's not defining a module
  * **Graydon**: why is it different?
  * **Niko**: well, good question
  * **Patrick**: much more flexible. you can declare an impl for any type, even one you don't own
  * **Patrick**: I feel like trait declarations and traitless impls are actually... fact that they're unified is making things confusing, leading to dreaded "cannot implement inherent methods for traits not defined in this crate" message
  * **Niko**: I never get that message
  * **Patrick**: if we cognitively separated them out I think people would hit them less often
  * **Brian**: I agree, they're different things
  * **Patrick**: that makes me think maybe mod for one, impl for other, but impl seems like a good name for both
  * **Patrick**: but I'd be ok with using mod for traitless impl. but if you did that then it's kind of dependent on mod being same name as type
  * **Graydon**: it's fine, I was just exploring the space
  * **Graydon**: I guess they really are different
  * **Patrick**: tantalizingly close, but yeah
  * **Patrick**: I can come up with a proposal
  * **Graydon**: caveat is try not to break phase distinction you already discovered requirement of in resolve

# Meta-discussion about region system

  * **Niko**: some design decision I'd like to put forward
  * **Niko**: don't know what's the best way to go about it; like one-shot closures, lots of subtle interactions and aspects to them
  * **Niko**: I'd kind of like to do a preso before group & then we can have a discussion where we're all on same footing. could write an email but that takes a lot of work
  * **Graydon**: I'm fine watching a preso, little more ad hoc back-and-forth
  * **Niko**: I'll try to draw up precise questions & work on a preso
  * **Graydon**: no slide deck, just notes
  * **Niko**: wanna do something to show. maybe notes and an etherpad
  * **Graydon**: yeah, that sounds fine

# License adjustment

  * **Graydon**: back-chatter conversation about license
  * **Graydon**: I said BSD originally, then said MIT b/c they're same in my mind
  * **Graydon**: these are the liberal licenses, more or less synonymous
  * **Graydon**: most modern form is Apache Source License 2
  * **Graydon**: for practical purposes identical to BSD
  * **Graydon**: profound enhancement is mutual patent non-aggression clause
  * **Graydon**: not really spiritual change to license, just patent protection. so I want to change Rust from MIT to ASL2
  * **Graydon**: don't think it will affect anyone, with one exception. if you're using a GPL2 or LGPL2 library that will link with rustc: in opinion of FSF not okay, because GPL2 doesn't protect you against patents. you're fine with v2 or later, can bump up to 3
  * **Graydon**: it's kind of a choose your poison situation. making incompat with past GPL stuff vs opening yourself up to patent trolls
  * **PaulS**: dual-license doesn't help you, right?
  * **Graydon**: don't know but strikes me as PITA. don't wanna do it
  * **PaulS**: people would only be able to use Rust under terms which involve patent non-aggression
  * **Graydon**: this was recommendation of our lawyers, after experience of updating MPL
  * **Dave**: just making sure: this doesn't say anything about stuff you compile with rustc, right?
  * **Graydon**: yeah yeah
  * **Brian**: what are consequences of people who want to write GPL2 software using Rust? they have to link to libstd?
  * **Patrick**: and rt
  * **PaulS**: we could put code to link against in public domain
  * **Graydon**: example of this in libgcc, they have an exemption for that
  * **Graydon**: we should do something the same
  * **Patrick**: we need to talk to legal about this, it's a very good point
  * **Graydon**: we'll make sure what we do is lawyer-approved
  * **Graydon**: since you all published under MIT we don't technically have to ask your permission, but to be polite I wanted to ask your permission
  * **Sully**: Mozilla probably owns the copyright anyway
  * **Brian**: one more question: do we know of any GPL2 IDE's that would not be able to link to rustc?
  * **Graydon**: don't actually know. my guess is there are a couple
  * **Dave**: emacs could just pipe to it, right?
  * **Graydon**: the legal status of that is not always clear
  * **Sully**: but emacs is GPL3
  * **Graydon**: it's a very good question, though. I'll dig into this
  * **Niko**: there's a wikipedia page with IDEs sorted by language and license

# Serialization of metadata

  * **Graydon**: Tim's been analyzing dependency structure of compiler, hairball of internal tables is causing grief
  * **Graydon**: on list of stuff to get done for stability: major issue is overhaulting metadata/import/export serialization
  * **Graydon**: switching serialization to something more future-proof
  * **Graydon**: tentatively explored Avro implementation in Rust
  * **Graydon**: that's a format with some good future, I think. but needs group decision
  * **Sully**: what advantages?
  * **Patrick**: don't have strong opinions, but whatever it is should be really simple
  * **Niko**: I don't care about format, but I want to inject one question: format is the lowest priority issue. real problem is hand-writing a bunch of code writing to a file instead of creating data structures to serialize
  * **Niko**: I would rather spend effort on rewriting metadata to be more future-proof
  * **Graydon**: phase 1 is to get metadata working using serialization
  * **Niko**: seems like a waste of time to build up really good Avro impl in Rust right now
  * **Graydon**: I'm concerned about EBML being crashy
  * **Niko**: are we crashing now?
  * **Graydon**: whenever you change data
  * **Niko**: that is solvable in other ways, but, that is true. it's actually not only EBML but even more specifically that we don't really use it the way we could. we do assertion against tags but don't really check them. we could do more tag checking and crash faster
  * **Niko**: we could isolate in a task, or rewrite to use result instead of failure
  * **Niko**: I think we should focus on that other stuff first, but Avro is a fine choice

# Unsafe exports in core

  * **Ben**: core exports a bunch of nasty stuff b/c std needs it
  * **Ben**: two bugs open: move uv to core b/c core exports priv
  * **Ben**: other says figure out how not to move arc and sync back to core
  * **Ben**: one option would be some mechanism for core to say exporting only to std
  * **Ben**: another option: not have such a mechanism, go into core and export stuff from core and put attribute to hide it
  * **Ben**: third option: export and document unsafe all over documentations
  * **Ben**: currently exported and not documented
  * **Ben**: thoughts?
  * **Graydon**: I have very very little opinion
  * **Graydon**: don't understand why it's a big concern
  * **Ben**: wanted some way to export only to core, people didn't like that
  * **Patrick**: just mark it unsafe
  * **Brian**: some things are *so* unsafe we don't want anyone to touch them
  * **Graydon**: unsafe is unsafe. we have unsafe stuff, we mark it as such

# Lint pass missing things

  * **Ben**: deprecated mode lint pass misses a couple things
  * **Niko**: let's talk after the meeting
