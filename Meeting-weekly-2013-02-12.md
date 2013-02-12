## Attending

jclements, tjc, nmatsakis, graydon, fklock, brson, dherman, pcwalton, azita

## Types that should not be sent

  * **Brian**: tasks doing I/O are bound to their schedulers so they can be single-threaded
  * **Brian**: on top of that, scheduler doesn't want to use managed boxes. in particular b/c scheduler isn't a task, doesn't have a local heap
  * **Brian**: way I'm thinking about doing I/O: little rc'ed type, scheduler and task have reference to it
  * **Brian**: problem here: rc'ed type shared between them is a type that is allowed to be sent, so it can escape from task and go to another thread. if you do I/O with it then it'll be a disaster
  * **Patrick**: and rc's will be messed up
  * **Brian**: there are this class of types that are not managed but shouldn't be allowed to leave a task
  * **Graydon**: a lot, or is this just an unsafe implementationy type?
  * **Brian**: this is the only use case I have, but forms basis for all I/O types
  * **Dave**: can't use privacy?
  * **Patrick**: can still send it
  * **Graydon**: yeah, ownership cuts across visibility
  * **Niko**: no reason we can't say you can tag arbitrary structs as non-sendable
  * **Patrick**: that's what I suggested yesterday
  * **Patrick**: Graydon has expressed desire for this in the past: rc'ed smart pointer type in stdlib for those who want rc & don't want tracing GC. that has similar constraints
  * **Graydon**: is there a difficulty with patrick's suggestion? trait called NotSendable
  * **Patrick**: could be an attribute
  * **Brian**: yeah, there's another problem: I/O types have to be ~ boxed
  * **Niko**: won't be able to send it or anything that owns it
  * **Brian**: ~ boxes are opaque. I mean ~ objects. the I/O types are abstracted to hide libuv implementation
  * **Graydon**: our object types can launder sendability?!
  * **Niko**: today ~fn and ~obj can only close over sendable things, so that wouldn't be a hole, but you wouldn't be able to put non-sendable into a ~obj. if we wanted to allow that we'd have to make it possible to close over non-sendable things and somehow reflect that in type. early versions of my proposal allowed it, we removed it
  * **Graydon**: heavy cognitive burden, specifying what it closes over & what it is
  * **Niko**: yeah. I guess that's a matter of defaults... doesn't erase cognitive burden but if normal thing is sendable maybe it's easier
  * **Patrick**: another possibility: could do on per-trait basis
  * **Patrick**: this trait is also non-sendable
  * **Dave**: how does that fix ~obj
  * **Niko**: that's true. if trait type is non-sendable, you can only close over non-sendable data if trait type you're encapsulating it in is itself non-sendable. won't work for ~fn but that may be ok
  * **Dave**: these are awfully ad hoc
  * **Patrick**: yeah, none of this is very clean at all
  * **Patrick**: clean way to do it, AFAICT, is: instead of attribute, it's a trait or kind, and you go back and add bound to trait or to ~fn, and you have these bounds
  * **Patrick**: we thought we could get away from them, but Brian is finding counterexamples that say perhaps we can't
  * **Patrick**: so that's the better-is-better solution
  * **Niko**: if we did that the simplest way, you'd write: `~fn:Owned` or `~fn:Send` ... hm, this is tricky because this is a negative kind. this data *is* owned, right...
  * **Patrick**: I was calling this "non-owned" yesterday. this is non-owned data.
  * **Brian**: we're doing the mem mgmt ourselves sort of
  * **Niko**: okay... that actually makes sense. where `@` says GC is doing it, this is saying "someone is managing the memory but you don't know who it is"
  * **Graydon**: can I look at this from a weird different direction? we have dynamic scoped variables supported by the language generally. one of the things we've talked about for dynamic variables in the first place is idea of making stdio and OS facilities be dynamic variables that are not exactly globals. right now they're pure globals, which is kind of a mistake. I wonder to what extent all I/O handles fall into that bucket. thinking of them as lifetime scoped and dynamic variables. in cases where not dynamic, allocated on stack, bound to current scheduler, and pass as parameters. if pass as parameters to annoying, make them dynamic variables. but doing in heap is perhaps missing point that they're task local
  * **Niko**: yeah, I was thinking something similar. imagine a special region called "task" -- I don't like special regions
  * **Graydon**: that's kind of the idea of TLS, they're task region bound. or at least the way I'd like to use it
  * **Patrick**: I'm a little nervous about taking file descriptors out of normal lifecycle management, nervous if you can't put fd in a ~box and if that goes away fd is closed
  * **Niko**: why does that have to be true? I don't feel like it has to be
  * **Patrick**: for rc'ed use case... I think that's an important use case
  * **Graydon**: when you say non-owned, can you repeat that concept?
  * **Patrick**: means data is not Owned, it can't be sent in particular, but it's not managed. it's not allocated in GC'ed heap
  * **Niko**: funny overlap between this and `*`. it's all those things except it can be sent
  * **Patrick**: difference is you can create safe interfaces around this
  * **Niko**: just wondering if you can unify with `*` somehow
  * **Niko**: if you had a special task lifetime it would sort of fall into this
  * **Patrick**: instead of `*` you make it `&task` -- think I like that better
  * **Graydon**: think I like that better
  * **Graydon**: when you impelment GC that's the local heap right there!
  * **Niko**: it doesn't sound unsafe enough to me, but we could find another name
  * **Dave**: `&unsafe task` -- done :)
  * **Patrick**: doesn't solve problem of putting in ~traits
  * **Niko**: they can have a region bound
  * **Patrick**: were we going to need a ~trait region bound?
  * **Niko**: ~fn do, it's very useful. I never knew of a use case for ~traits
  * **Patrick**: how notated?
  * **Niko**: `~'foo fn(...)`
  * **Patrick**: how does this solve brian's use case? how's it fit together?
  * **Niko**: objects would have to be something like `~'task Reader` -- not a clear picture of what this would mean yet
  * **Graydon**: I think it would be an area you'd put in TLS. that's my vote
  * **Patrick**: needs to be a special kind of arena where you can deallocate
  * **Graydon**: an unsafe type that works like a heap that we call an arena. call it what you like
  * **Patrick**: ok, just right now arenas don't let you deallocate
  * **Niko**: I guess heap is really the right name
  * **Patrick**: sounding more plausible to me
  * **Niko**: okay, that's sounding more built-in...
  * **Patrick**: type system doesn't care about it. it just cares about a marker that you can't send it
  * **Brian**: doesn't really matter where the region is, as long as it has the annotation
  * **Niko**: some unsafe casting function to transmute?
  * **Patrick**: yeah, and ultimately you'll wrap that in unsafe wrappers somehow
  * **Niko**: ok, and such a borrowed pointer is not sendable
  * **Graydon**: who declares existence of lifetime? how's it come into existence?
  * **Patrick**: transmute to it
  * **Niko**: I think it has to be static, like an ever-present lifetime name that's built-in. smaller than static but bigger than every function
  * **Brian**: sounds like a region for the local heap. is there one for the exchange heap?
  * **Niko**: ~ :) it's not a lifetime because it's not block-scoped in anyway
  * **Brian**: ah ok
  * **Brian**: well neither is local heap
  * **Niko**: well this isn't a *real* lifetime, but we're making it like one
  * **Patrick**: shoehorning ref-counting into this scheme is gonna be really strange
  * **Patrick**: suppose I want to make a rc'ed smart pointer that can't leave task. how?
  * **Niko**: allocate memory somehow, transmute pointer to `&'task` and wrap in new type
  * **Patrick**: that's two levels of indirection
  * **Patrick**: struct RC<t> should just have a uint and t
  * **Niko**: you're not going to pass it by value! you have to point to it
  * **Niko**: `struct RC<T>(&'task RCRepr<T>); struct RCRepr<T> { ref_count: uint, data: T }`
  * **Patrick**: oh right! I'm cool with this now
  * **Brian**: that means you can create one of these that wouldn't be bound by that lifetime, which is unsafe
  * **Patrick**: not unsafe, just stupid
  * **Brian**: if you create a RC type that isn't a task-borrowed pointer, that'd be unsafe
  * **John**: patrick's saying counter just wouldn't be used
  * **Brian**: but you could send it!
  * **Patrick**: here's the thing: to create multiple references, you have to clone it. we can say .clone() only works on `&'task`
  * **Niko**: not about clone method, just the RC wrapper is a struct. if you copy you copy the whole struct. you need a layer of indirection to have aliases. to get that you have to use a safe aliasing mechanisms, or `&'task` which can't be sent
  * **Brian**: I understood patrick's explanation better
  * **Niko**: don't know why you'd need a .clone() ...
  * **Dave**: are you talking about special-casing the name .clone()?
  * **Patrick**: no, just put a bound on it
  * **Niko**: ok, patrick's explanation was better than mine
  * **Patrick**: hm, does this want to be a non-copyable type? good point
  * **Niko**: feeling like Yet Another pointer type, which I don't like
  * **Patrick**: we could see if * being non-sendable breaks stuff
  * **Brian**: if you want to opt in to sendability transmute it to an int?
  * **Patrick**: yeah or a newtype int
  * **Niko**: we could get rid of * altogether, and do some newtypes
  * **Patrick**: then how do you annotate that something is non-sendable?
  * **Niko**: exactly, still need a way to put it in a box. so ignore region idea, go with *. but we still need a way to put it in an object ==> bounds. if we have bounds, we could get rid of * altogether
  * **Niko**: go back to original scheme where we can tag somethign as....
  * **Patrick**: this is back to square 1
  * **Niko**: I'm just excited about idea of only having 3 pointers
  * **Graydon**: I'm less excited about that
  * **Niko**: it's orthogonal sort of
  * **Patrick**: totally
  * **Niko**: well not totally, we're now talking about 5 pointers (!) as only alternative!
  * **Patrick**: could go back to solution based on kinds, either do with attribute on structs or with a trait
  * **Niko**: then we just need these bounds
  * **Patrick**: could work around it with some sort of defaults
  * **Niko**: original plan was if you wrote `~fn` or `~trait` without bound you got Owned by default
  * **Niko**: but if you wrote a colon (e.g., `~fn:` or `~fn:Owned` etc) you get what you wrote
  * **Patrick**: only people who will use bounds are Brian and... potentially people writing a lot of smart pointers
  * **Patrick**: people who want to work around tracing GC
  * **Graydon**: slightly orthogonal question: on a cognitive complexity basis for implementor, assuming we can hide this in presentation of the language, how bad is it from compiler perspective to have the bound there?
  * **Niko**: not a big deal. it's part of the type, but... today we have basically hard-coded bounds; makes it a little less hard-coded. not a big deal. took it out not for impl complexity but b/c it seemed we didn't need it
  * **Patrick**: this was part of his proposal all along, dates back to last summer
  * **Graydon**: yeah, seen it before, thought we got rid of it for simplicity. just wondering how much is for user and how much for the compiler
  * **Patrick**: having bounds is more orthogonal from a theory perspective, and kind of from an implementation perspective... but probably more code.
  * **Niko**: but not a lot more
  * **Patrick**: doesn't affect theoretical complexity; we'd be replacing some default with customizable defaults. pick your poison. from user's perspective it is more complexity; that's the danger
  * **Patrick**: I don't see alternatives that aren't ad hoc or creating a new pointer type
  * **Graydon**: if we think of an I/O handle as somethign that's sendable but it's a TLS key, if it snaps into existence in your task, it's yours.
  * **Graydon**: you can put file descriptors in heap, but closing themselves if they go out of scope -- if they get sent to another task it has no effect
  * **Brian**: oh, I didn't understand that. you'd make it storable by not storing pointer. you'd store number that lets you get the pointer out of TLS
  * **Graydon**: either way. if you store the pointer you'd have to have a new kind of lifetime
  * **Niko**: or push it on user to trace that lifetime through. type system can express that but will require more annotation on part of user who wants to put one of these things into... seems like I/O is not such an exotic use case
  * **Brian**: if we're using borrowed pointers I don't see why it requires anything new. just synthesize a borrowed pointer in your stack frame. no new task lifetime or anything?
  * **Niko**: as long as you're willing to say you have to get these through a borrowed pointer. point was avoiding a lifetime parameter on everything
  * **Brian**: that is a limitation
  * **Niko**: seems probably too big
  * **Patrick**: I'm nervous about that
  * **Niko**: that's where uint fd comes in
  * **Graydon**: reason I have some affinity for that: multiple tasks reaching for global fd's is a usability hazard in first place. strong argument that things that are global ought to be dynamic variables anyway
  * **Graydon**: we allow you to reach into env, OS, etc... not implausible those things should actually be one level indirected dynamic variables. you can mock OS, I/O, etc. no way to do that right now
  * **Dave**: I can imagine doing this for security isolation
  * **Graydon**: even just mocking and testing
  * **Graydon**: every library user wants to be able to reach for I/O conveniently. like random numbers, timers, environment. and yet, that's in tension with replacing them. that's what dynamic variables are for
  * **Patrick**: I don't understand what's being proposed here, but I'll take it offline
  * **Niko**: utility services shouldn't be statically linked
  * **Patrick**: I understand, just don't understand lifetime of fd's here
  * **Graydon**: I'm not sure
  * **Graydon**: unix process model says these things have process-oriented lifecycle, but we're introducing a new intraprocess abstraction
  * **Patrick**: problem is unix lets you leak them easily b/c you ahve to call close
  * **Patrick**: nice thing of tying to memory in C++ is it's much harder to leak them. nervous about giving that up
  * **Niko**: struct with private field and destructor?
  * **Patrick**: make sure doesn't go to other tasks...
  * **Niko**: still leakable, just not easy
  * **Niko**: if you go uint route, you don't get an airtight solution, that's the bottom line
  * **Patrick**: I'm nervous about going into an I/O design where we discover you get leaks and then we have to go redo it
  * **Graydon**: good concern, yep.

## MIPS

  * **Patrick**: we have MIPS.
  * **Brian**: yesterday while Graydon was out, wanted to merge MIPS so Graydon couldn't object, but I once again realized that merging MIPS is gonna cause a ton of churn; everyone's gonna have to rebuild LLVM
  * **Brian**: figured I should get actual approval to make the merge. just gonna be a code drop, not gonna be able to test it
  * **Graydon**: I defer to your opinion. weird maintenance burden...
  * **Patrick**: discussion about tiers we were gonna inevitably have at some point
  * **Patrick**: I think if we accept MIPS we're gonna have two tiers. clearly can't do maintenance & bots for MIPS
  * **Patrick**: are we okay with two tiers?
  * **Patrick**: currently all platforms are tier 1
  * **Niko**: can we put BSD tier 2?
  * **Patrick**: no it's tier 1
  * **Niko**: it is *now* but I don't *want* those bots
  * **Patrick**: because we have 10 minutes I'd like to table the BSD question and just talk about should we have tiers
  * **Patrick**: I like tiers, I think they're inevitable
  * **Graydon**: I think they're inevitable from a releng perspective. we'll end up doing triage that drops things
  * **John**: how do you publicize the tiers?
  * **Patrick**: just goes in the docs
  * **Brian**: we do say FreeBSD is unsupported
  * **Patrick**: in dev process in practice it's acting as tier 1
  * **Graydon**: interesting division here... I'm pretty sure ARM is gonna elevate itself, maybe ARM Android, to tier 1. don't expect we'll be maintaining snapshots
  * **Patrick**: tier 1 host vs tier 1 cross? host means build on it, tier 1 cross means target it
  * **Graydon**: if you break it, back you out
  * **Graydon**: tier 2 means dubious quality
  * **Patrick**: nobody really compiles on ARM... well some people with raspberry pi do
  * **Graydon**: but generally cross-compiled
  * **Patrick**: MIPS is same way. don't see a whole lot of point in supporting compiler for those architectures
  * **Patrick**: we can say tier 1 target + tier 2 host for those platforms
  * **Graydon**: curious specifically about MIPS: why? I was really surprised to see in the queue
  * **Patrick**: we were all very surprised
  * **Niko**: I was wondering same thing
  * **Graydon**: intended for routers? only case I know of
  * **Patrick**: TV's
  * **Brian**: huge in China. they have their own processors they're building
  * **Dave**: who contributed it?
  * **Patrick**: crabtw
  * **Brian**: also did the FreeBSD
  * **Graydon**: and the amazing port of the x64 ABI rules
  * **Niko**: for which we are eternally grateful
  * <3 all around
  * **Graydon**: don't know if we'll have an emulator for MIPS any time soon
  * **Graydon**: so can we land this and deal with the fallout for LLVM breakage?
  * **Brian**: I can do over weekend
  * **Brian**: what intervention is gonna be required?
  * **Graydon**: in theory, activating every one of the builders with CLEAN-LLVM=1 parameters
  * **Graydon**: reason last time had so much fallout was: we were doing wipe, which was doing something funny... somehow resetting... was deleting workspace and causing that to revert to previous form somehow
