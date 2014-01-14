# Agenda 01/14/2013

* static trait method syntax
* iterator renamings (acrichto) https://github.com/mozilla/rust/pull/11001
  * list of renamings: https://github.com/mozilla/rust/pull/11001#issuecomment-31571205
* default type parameters (acrichto) https://github.com/mozilla/rust/pull/11217
* lang items for primitive impls (acrichto) https://github.com/mozilla/rust/pull/11409
* fmt::Default => fmt::Format (acrichto) https://github.com/mozilla/rust/pull/11298
* no_mangle for extern fns (acrichto) https://github.com/mozilla/rust/issues/11089
* crate_type = "lib" (acrichto) https://github.com/mozilla/rust/issues/11253
* foo_opt => foo (acrichto) https://github.com/mozilla/rust/pull/11129
* moving from statics (nmatsakis) #10577
* env pointer (nmatsakis)
* friend of the tree (brson)
* 1.0 language feature freeze (brson)
* invitations for work week (pnkfelix)

# Status

- brson - diagnostics, android, channel proposal, 1.0 roadmap
- acrichto - mutexes, upgrading channels, rustdoc improvements, leaking traits resolve bug
- pnkfelix - GC (dbaupp's, and bdw hackery)
- nmatsakis: temporary lifetimes (almost, ALMOST building!)
- pcwalton: removing @, getting backlogged servo patches landed

## Friend of the tree

- brson: Eduard Burtescu (eddyb) has been contributing since October. A lot has been on the compiler, including trans. He has reduced rustc memory usage, optimized vector operations, removed deprecated features, and cleaned up ancient code. Finally, he cleaned up our misuse of the environment structure to pass the self param.

## Static method on a trait

- acrichto: Unless you're calling on a type parameter, nobody does it. But it's easy to do. 
- pcwalton: Keep the original syntax. But, should add something that allows you to do T::whatever or Int::whatever. Now, should search like the . operator.
- nmatsakis: trait:: would be the same as recv. in terms of the algorithm.
- acrichto: SUpport Trait::?
- nmatsakis: It would disambiguate.
- pnkfelix: Could explicitly write out the pair .
- nmatsakis: `for` keyword there: `Trait::<A, B for  C>::...`
- acrichto: Didn't see an issue on this.
- brson: Try to find and nominate it... for 1.0?
- pcwalton: No, it's backwards-compatible.
- nmatsakis: If we add the explicit form we can add the nicer form whenever we want.

## Environment pointer

- nmatsakis: Question! Working on a branch to remove the environment pointer altogether for bare functions. Historically opposed to it. We have an extra pointer (env) on all of our functions. It's only used if the function is a closure. We have this because it makes it easier to coerce bare functions into closures by passing a NULL pointer as the environment. OTOH, it's an extra argument. That makes the calling convention less C-compatible. In particular, we put the env pointer at the beginning of the list, which gives it a good register, etc. eddyb's working on removing it altogether. I'd been thinking of moving it to the end of the list. Thoughts?
- brson: Moving to the last argument seems reasonable.
- acrichto: How often do we coerce bare functions to closures?
- nmatsakis: fairly regularly. I know because when I break it in the typechecker it fails all over.
- pcwalton: Would we be c-compatible if we move it to the end?
- nmatsakis: Need to apply all ABI rules. If we did that, then yes.
- pcwalton: Would be nice.
- brson: But you can already get C ABI support via FFI syntax.
- nmatsakis: I also don't know if I'd want to guarantee it. We might want something faster.
- brson: How does this stuff interact with methods as first-class functions?
- nmatsakis: Pretty much orthogonal. But it makes methods and first-class functions exactly identical. I think we could do that and have an extra environment pointer (which is a great thing). One clarification (patrick). Move it to the end; I don't know if that's compatible or not.
- pcwalton: Since people stuff extra arguments on main(), I assume it is. Like var_args.
- brson: Compile-time expense? Or a runtime effect for the coercion?
- nmatsakis: Introduces a trampoline. You get a generic function that bounces through to add in the environment in the actual call. So, it's being used for all bare functions. If you have lots of them, you'll probably have bad CPU prediction behavior. Complication with procs because the drop_glue tries to free the environment, which crashes. Have to take that away or find some solution.
- brson: Nothing from having a trampoline-per-coercion?
- nmatsakis: Yes. But you'd still be introducing an extra virtual function call. If you coerce from a named function, you can make a wrapper. The only case that's hard is when you have a bare function pointer to a closure pointer. That's probably a corner case. It's possible we could just disallow that. I imagine most of the time you're coercing from an actual function.
- brson: Difference between actual and bare?
- nmatsakis: Name of a global function definition vs. an argument. e.g. the named function itself is statically known. Then we could have a wrapper.
- brson: Thoughts? Well, aiming for C ABI is almost a bad thing because it handcuffs us. Introducing an inefficiency to get closer to the C ABI seems like a waste to me.
- pcwalton: Isn't it usually a win to put the environment pointer at the end for locality?
- nmatsakis: He removes it altogether. But I do think that's better.
- brson: Even if we have that pointer, that doesn't make us incompatible with the C ABI, it's just a void * argument.
- acrichto: Doesn't a method with a self transmute from the environment pointer?
- nmatsakis: Not any more. For methods, the 0th is the out, the 1st is { X OR, Y OR self). We want the self to be the first argument in the list. And want the environment to only be there for closures.
- acrichto: Removing the transmute should open up some optimizations...
- nmatsakis: Yes, but it's gone already.
- brson: Opinions?
- nmatsakis: I remembered that this trampoline comes up more often than I'd thought. I'd like to see a persuasive argument for why the environment pointer is doing us harm. e.g. that it prevents an LLVM optimization, increases register pressure, etc.
- pcwalton: I don't see any harm with having it at the end. Then we could (at the call site) not push that argument. Not pushing is the same as having garbage values there.
- nmatsakis: Or undefined?
- pcwalton: We could just optimize the case in LLVM & then it would be the same as C/C++ and we could keep the same environment. For register-based, just don't load anything into the register. Even on the stack, we could just not push something.
- nmatsakis: LLVM probably doesn't do anything today... adjusts stack in the beginning... we should compare the machine code.
- brson: Niko, can you own this and work with eddyb?

## Language feature freeze for 1.0

- brson: Mentioned multiple times that people are worried about random changes to the language in PRs that we keep debating. Seems like a source of ongoing maintenance burden and instability. Should we officially have a feature freeze?
- pcwalton: Maybe be "slushy" instead of frozen? A feature freeze would mean not changing the borrow checker, etc. We still haven't proved all this safe. So we can't freeze until we have safety. But, we can institute "no new features outside target areas."
- nmatsakis: What concerns me is PRs for well-intentioned things like variadic generics that are big changes...
- pcwalton: Those specifically I want to talk about because they'd be backwards incompatible (they interact with allocators), and since we want an allocator story, we might want to address that.
- nmatsakis: Okay. I'm in favor of limiting our scope.
- larsberg: Maybe just require an issue first before a PR?
- brson: We do have a process for that.
- nmatsakis: I think we have one. We've been pretty light on spec / code stuff. We're as guilty as anyone else, but it might be helpful. 
- brson: Any other thoughts? I don't hear a lot of fiery enthusiasm.
- acrichto: Maybe have the way you do it be official.
- brson: RFC process?
- acrichito: Just have roadblocks?
- brson: Concerns with an improvement process for Rust is I just don't want to be distracted from shipping 1.0.
- jack: I don't remember us sending mail to the mailing list with the goals and what's in/out. If we ask the community to be more "slushy" that would probably do it.
- brson: Need a roadmap. Emphasize that we're working on some things and don't want to be working on others, that may do it. If we were going to have a feature freeze, we'd at least have to explain what isn't frozen. We will do nothing now.

## Iterators

- acrichto: 11001 renames a lot of iterators. There's a great list in the PR itself. It's inconsistent if you have iter, iterator, or nothing and renames it to THING or THING+elements. You can look through the list, but the suffix of iter or iterator is gone. If it's returning a noun then the iterator is the plural of the noun. This is mainly something we need to make a decision on. I'm not moved one way or the other myself.
- nmatsakis: Like consistency. Not sure I like the lack of the word iterator.
- brson: Could tack iterator onto all of these new names and it'd still be consistent. Hrm. PortIterator is Messages?
- pcwalton: I like the lack of the name iterator. Python used to have this convention of putting X on the front, so they got rid of it in Python 3. I have a feeling that's a useful precedent. You almost always want the iterator. If you want the other thing, say .collect.
- acrichto: We have a lot of them. What's the naming convention? Here if it's a thing and the noun describes it, pluralize. But not in all cases. If it's a Set and you're moving elements, it's a SetIterator... lot of edge cases. 
- nmatsakis: I was reading the PR and I like it. Better than what we have now.
- brson: Not enterprisey, which I like. Items is shorter than Elements, so we could use that instead since we like it.
- pcwalton: I like items better than elts because I dislike abbreviations and favor of short words. So +1 for items.
- jack: I'm fine with Elements or Items.
- acrichto: Concerns raised on Items. Something about... hrm, not sure.
- brson: Seems OK. Land as-is?
- acrichto: Would take everything without Elements as-is. I'm ambivalent about it, but...
- pnkfelix: Items is better. And starts with "it" like "it"erator
- brson: acrichto - can you land this and write up the convention on our style guide? 
- acrichto: Switch to Items but otherwise take as-is.

## Moving from statics

- nmatsakis: We currently have bugs when you make use of a static item whose type has a destructor. It's not entirely obvious how we should fix this. An example is if you have a static item whose types is an optional owned pointer. Then, you could initialize it with None, but when you use it, the current code in the compiler then tries to null out static memory because of how we do frees and that gives us a crash. At first, thought we should just not try to null that memory. But, on the other hand, if we have types like structs with a destructor, then that's weird because it should run, but how many times? I guess, I want to talk about it, but don't know what to say. Opinions? Or should I try to write up a proposal? We need to fix it.
- acrichto: Weren't we going to disallow destructors and statics?
- nmatsakis: I think we can't disallow anything whose type involves a destructor. Though we could disallow any expressions that create a value that needs to be destructed. This option example is pretty good. We have to have None hanging around and it's kinda static. Maybe those are just different.
- brson: Mutable statics?
- nmatsakis: Can't have a generic static.
- pcwalton: Allowing statics with destructors, in C++, they run after main exits in an unspecified order, blocking shutdown.
- nmatsakis: Not that!
- pcwalton: I think we'll be forced into that if we allow it. People will say, "you're doing the wrong thing" if we don't disallow it. Then we're stuck with life after main.
- nmatsakis: Also, I hate static muts. We could say all statics have types that don't have destructors. Might rule out JS Class stuff with initializers that have optional things that are null. 
- pcwalton: Could just have separate type for that kind of stuff.
- acrichto: If you have a type with a destructor, then you should have a static. Doesn't fix noncopyable statics. Should be handling that with a type-something. So, still have the problem of moving out of statics because we don't want to zero it out.
- nmatsakis: I'm using destructor as shorthand for anything non-copyable. Have to replace those atomics with a function that returns a new one?
- acrichto: Should allow non-copyable statics (e.g. for the atomic counter case). If we have a noncopyable attribute, type, or member, that will create the distinction. Should definitely give a compiler error about moving out of statics. But you can initialize other statics with the same bit-pattern.
- nmatsakis: What can you do with it that's useful?
- acrichto: Take its address. Primary use case is for having a bit pattern for initializing other variables with. Same thing with a static mutex. Want to let people copy it to their bit pattern, but nobody should be able to move out of it.
- nmatsakis: Can't have a type that owns something with a destructor in statics?
- acrichto: Yes. Both mutable and immutable.
- nmatsakis: Reasonable. And disallow moves out. May consider enum variants as rvalues. If you reference None, we don't point you at the global; we create a new one.
- pcwalton: Agreed.

## Workweek

- pnkfelix: Is it scheduled yet?
- brson: No. Submitted the form.
- azita: Still boston?
- brson: Unknown.

## Docs on primitives

- acrichto: We have no docs on the prims. Adds a lang item per type. Then, the docs on that impl become the docs on the type. You get a nicely linked-up thing for stuff like the u64 pages. Lots of people have commented that we should have a shadow `struct u64` and then lang item it. What are people's thoughts? It's a real problem that needs to be solved. And my way doesn't handle strings, vectors, or tuples. Any thoughts on if this is the best way to go about it or not?
- nmatsakis: I don't have a strong opinion. I need to look at the PR.
- acrichto: 11409. We can worry about it for the next meeting, since pcwalton and brson are both gone.
- nmatsakis: Yes.
- nrc: Purely for documentation purposes?
- acrichto: Yes, lots of one-shot traits (since we only have traits for primitive types). So if you want to realize you can call Add on an integer, you have to find the Trait and then see the documentation. So, like CheckedAdd, etc. are important. 
- nmatsakis: So, now the methods are inherent instead of Traits?
- acrichto: I made changes to coherence so that the one lang item will belong in libstd and rustdoc knows how to track it down.
- nmatsakis: Seems OK. A shadow struct type seems bad.
- acrichto: Not sure how to do vectors. Problem with strings is &str and ~str split. Both string type methods. with vectors you have &vec, ~vec, etc. It's just tough to see what's all possible that you can do on those types.
- pnkfelix: Non-starter to have rustdoc just do all the work to gather up the docs?
- acrichto: Nothing has the knowledge of the "standard crate" for primitives, etc. Every generated doc would have to have the primitives section that you would then look at.
- nmatsakis: Yeah, let's discuss it later. It doesn't seem bad.
- acrichto: I'll think more on it.
