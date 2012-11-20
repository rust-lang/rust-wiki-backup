## Attending

tjc, pnkfelix, graydon, nmatsakis, dherman, brson

## deriving

  * **Patrick**: started implemented with macros, seems to be going well; can probably get it all done today
  * **Patrick**: currently works with Eq on structs and C-like enums; just works -- seems easier to do with macros
  * **Patrick**: so may be right approach, and avoids building in compiler smarts e.g. type-checking rules & special trans code
  * **Patrick**: now need to get working with n-ary variants & type-parameterized stuff, also needs to work on IterBytes as well as Eq and Ord
  * **Patrick**: going pretty well, hoping I can get it done today
  * **Brian**: what's status of new quasiquoter?
  * **Graydon**: landed & enabled in snapshot, bootstrapping issues; done, just needs to replace all the other stuff with it & integrate to avoid messing up bootstrapping cycle
  * **Patrick**: deriving is not using quasiquoter
  * **Graydon**: using explicit syntax extension, not macro?
  * **Patrick**: yeah. generating AST's by hand
  * **Brian**: pretty complicated
  * **Patrick**: yeah, it's not pretty code. but not that much code. don't expect more than 1KLOC. tried to comment it well
  * **Brian**: hard to read; constructs code bottom-up, so hard to follow
  * **Patrick**: yeah
  * **Brian**: just gonna get worse
  * **Graydon**: I'll chat w/ you on IRC about switching to quasiquote ASAP. hopefully today or tomorrow. very nearly done
  * **Patrick**: is everyone okay with this direction? getting rid of deriving as a special type-check + trans thing and just moving it over to a macro
  * **Graydon**: fine with it
  * **Patrick**: attribute actually
  * **Niko**: thumbs up
  * **Dave**: seems pretty natural, yeah

## @mut [T]

  * **Patrick**: minor change. since deprecating mut inside in vectors: [mut T] and switching to mut [T]
  * **Patrick**: moving towards mut as qualifier on owner or fields
  * **Patrick**: someone asked in IRC how to make a mutable @vec. seemed strange that that's @[mut T] when we're moving away from that generally. seems more natural to do @mut[T]
  * **Graydon**: agree. legacy syntax. aim for consistency. should be @mut[T]
  * **Brian**: same for twiddle?
  * **Patrick**: I thought twiddle...
  * **Brian**: why not?
  * **Patrick**: because leads to more choices when you make data structures.
  * **Niko**: I'm wondering whether mut should be removed from owned content, like fields and ~mut
  * **Patrick**: separate discussion I didn't want to get into here
  * **Niko**: depending on that discussion, ~mut [T] would be legal depending on that decision
  * **Graydon**: I agree with niko on that
  * **Patrick**: that is true
  * **Patrick**: ok, basically sounds like agreement. move mut outside so it doesn't confuse people as much
  * **Graydon**: if that concept continues to exist. if mut after @ or ~ continues to exist, yes.
  * **Patrick**: I think after @ will always continue to exist. what Niko was talking about basically means mutability moves to owner
  * **Patrick**: owner is either ultimately local stack variable or garbage collector
  * **Dave**: @ specifies owner to be the GC
  * **Patrick**: right, so that's where the mut goes
  * **Graydon**: ok, that's consistent. takes a little explaining, but less than yet another special case
  * **Patrick**: all a little weird, but whole reason we're doing this is to make freezing work. freezing is property of owner, so owner can change things
  * **Graydon**: that's fine

## impls of typedefs

  * **Patrick**: weird question bjz keeps hitting
  * **Patrick**: haven't decided what we want, so compiler does random things
  * **Patrick**: when you impl a typedef
  * **Patrick**: don't really know what to do here
  * **Patrick**: just food for thought for now, I guess
  * **Patrick**: can make impls on types, can associate static methods with them
  * **Patrick**: if you do that on a typedef, non-static methods not too problematic
  * **Patrick**: what about static methods, though
  * **Patrick**: should they be scoped to original type or the typedef type?
  * **Patrick**: now that I say that it seems obv it should be the original
  * **Patrick**: but bjz had some useful patterns where he wanted them to be scoped to the typedef
  * **Niko**: think this ties into how much resolve and typedefs interact. we kind of discussed before but didn't write something up. if you implemented typedef with pub use, then it would all kind of make sense and work
  * **Patrick**: lemme describe what bjz was trying to do: wanted convenient typedefs of his types with type parameters filled out. e.g., vec2<float>, typedefed vec2f to it. started putting static methods on vec2f. this forwarded to vec2<float>. maybe this isn't appropriate forum for this discussion
  * **Tim**: in Haskell can't do that w/o ghc extension flag, b/c it makes overlapping instances more complicated. so there's precedent
  * **Patrick**: also ties more generally into: when you call static method, sometimes you want to specify self-type explicitly, no way to do that now w/o type-annotated let statement
  * **Patrick**: this is all kind of out of scope, let's table it
  * **Graydon**: one thing I care about: don't add contortions unnecessarily

## bounded type parameters on structs

  * **Patrick**: right now, compiler lets you say `struct Foo<T:Bar>` -- lets you type it, parses it, does nothing with the bounds
  * **Patrick**: didn't even realize this was possible until burg tried it and it didn't work
  * **Patrick**: I think there's possibly a reasonable semantics, haven't thought about what it is. it's hard to implement b/c of destructors
  * **Patrick**: he was surprised that if you do this, you want to call method of T on destructor, but there's no place to store the vtable. we just ICE'd (internal compiler error)
  * **Patrick**: got to trans and then freaked out (said: "the impossible happened", \o/)
  * **Patrick**: propose forbidding these declarations. type parameterized structs and enums must not have bounds on parameters
  * **Patrick**: can still create impl on that struct with bounds
  * **Patrick**: could make all methods depend on impl bounds
  * **Graydon**: reasonable restriction. points to what's going on with how our OO works: we're giving a separate treatment to data and functions.
  * **Patrick**: there is a workaround for what he wanted to do: use these objects, trait value types. cast to trait, put it in your object as a first-class type
  * **Graydon**: why did he do that in the first place?
  * **Patrick**: complicated thing in wrapping core foundation types. have types that in theory could have diff types of destructors, want to wrap them... basically, dealing with wrapping external resources. was trying to make them type safe in rust. wrapper wrapped actual core foundation type as one of type parameters. but in practice wasn't necessary. kind of a COM system; they all have the same destructor (Release)
  * **Graydon**: will this use case go away when destructors are fully implemented?
  * **Patrick**: don't think so. that doesn't change machine-level rep of destructor
  * **Patrick**: have to be able to call destructors in virtual way. no way to monomorphise
  * **Patrick**: haven't thought through this very hard... it's a question of compilation. all the Drop trait does is change how we type-check destructor, not how it's implemented
  * **Graydon**: this seems very destructor-specific problem
  * **Patrick**: yes, it is
  * **Graydon**: that's the only implied method that's connected to data value itself rather than impl associated with it
  * **Patrick**: there is indeed question of whether Drop trait is right way to do destructors. Niko had concerns there, b/c it is kind of separate, tied to data itself
  * **Patrick**: that code is still there; we could go back on that decision
  * **Niko**: not sure. seems diff from other traits. kind of ok to have a special trait with special rules. maybe still a nice way to present the idea
  * **Patrick**: just depneds on what's simpler. simpler by having new construct or simpler by same construct with special rules
  * **Graydon**: maybe not the right place for this conversation. but let's talk about the implementation of destructors
  * **Niko**: couple quick thoughts: I think there is a reasonable semantics to putting bounds on struct declaration, if you want it to guarantee. problem with destructors seems mostly a plumbing problem (implementation, not theory). that said, seems like a pure extension to add. if it makes our lives easier to remove for now, we can add it later. no back-compat problem
  * **Patrick**: in C++ if you want destructor overridden, have to have virtual destructor. we don't have concept of virtual destructor. all destructors are non-virtual. what I was a little concerned about was: certain combinations of generics make it so you don't statically know what bounds of type parameters at time you need to generate destructor
  * **Niko**: I don't believe that is true; if you boxed a value that has destructor, we'll have to package as part of type descriptor the destructor. this is also true for copying
  * **Niko**: we can discuss later
  * **Niko**: I feel totally happy saying we're gonna make this rule for now b/c it makes our lives easier
  * **Graydon**: unprohibit later when we can make the semantics sensible
  * **Patrick**: yeah
  * **Patrick**: will have to thread a fair amount more stuff through trans to make it work

## unsafe pointer assignability

  * **Patrick**: unsafe pointer as supertype of region pointer causes problem b/c of overlapping instances
  * **Patrick**: core defines certain things on pointers that it does not define on region pointers: Eq
  * **Patrick**: Eq for region pointers auto-dereferences region pointer and does structural equality
  * **Patrick**: doing that on unsafe pointers is really questionable
  * **Patrick**: Eq is not tagged unsafe. if it dereferenced an unsafe pointer then safe code could derefrence unsafe pointers by calling ==
  * **Patrick**: I think Eq needs to be pointer equality for unsafe pointers
  * **Patrick**: but instances are difference so that's an overlapping instances error
  * **Patrick**: so subtyping doesn't really work
  * **Patrick**: one possibility is to use assignability rules, so can coerce region pointer to unsafe
  * **Graydon**: how often do we need to do this automatically? we have casting
  * **Patrick**: we write to_unsafe_ptr a lot, pretty annoying
  * **Niko**: using assignability seems pretty harmless. you know you're calling a C function
  * **Graydon**: yeah, casting every parameter on a C function gets a little stale
  * **Brian**: don't have an opinion, but haven't felt that pain. don't mind writing the coercion
  * **Patrick**: could also just not do it
  * **Patrick**: put it on backburner
  * **Graydon**: I think it's okay to do assignability. if there's a problem we'll discover it
  * **Graydon**: I say go ahead and do it

## remove as ~Trait

  * **Patrick**: could possibly use assignability here too
  * **Patrick**: doesn't involve allocation of a box, all it does is package something up in pair of vtable and pointer
  * **Patrick**: triple in case of twiddle
  * **Graydon**: thing in question already has to be stored the right way
  * **Patrick**: yes, does now
  * **Patrick**: so we can just make the assignability work, makes people's lives a little easier
  * **Patrick**: very cheap operation at runtime, so doesn't affect perf
  * **Niko**: I'm a big fan of this
  * **Graydon**: yeah, storing a pointer and a constant into a pair
  * **Brian**: does this mean two types of implicit conversion? borrowing and converting to objects
  * **Patrick**: yes
  * **Patrick**: maybe unsafe pointer too
  * **Brian**: just want to keep track of them all
  * **Patrick**: yes, definitely coercions are scary. barrier they have to meet is high
  * **Graydon**: barrier is has to be something psychologically equivalent to subtyping. thing you're coercing to is harmless subset of functionality. just a restriction
  * **Dave**: and has to be cheap
  * **Graydon**: yeah
  * **Patrick**: so numeric conversions don't fit, because they're changing the semantics in observable ways

## cross-crate constants

  * **Patrick**: discussed like a year ago. rejected b/c we didn't have cross-crate inlining
  * **Patrick**: now we do! seems like less of an issue
  * **Graydon**: last time my feet were dug in about strict rules about versioning. we've torpedoed those assumptions. everything's versioned simultaneously now
  * **Graydon**: but cross-crate constants aren't gonna make things any worse. exactly the same as inline functions
  * **Patrick**: everyone ok?
  * **Graydon**: only reason they're currently prohibited is we don't have the machinery. there's even a comment saying go ahead and implement
