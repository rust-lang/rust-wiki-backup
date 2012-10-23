## Attending

graydon, brson, tjc, dherman, pcwalton, nmatsakis (via chat)

## Dynamic loading

  * **Brian**: bjz is building OpenGL bindings; world of OpenGL is pretty crufty; right now he's defining API's for all levels of OpenGL, trying to use cfg to compile out the ones unavailable on current platform
  * **Brian**: most these days tend to use dynamic binding
  * **Patrick**: Mozilla doesn't directly link against anything; load library and GetProcAddress every symbol
  * **Patrick**: I think that's pretty common for OpenGL that's cross-platform; you never know what symbols are going to be available
  * **Patrick**: e.g., what I did with ES was: download header, bindgen'ed it, commented out stuff not available on my mac
  * **Patrick**: you need extensions for good perf; extensions vary by platform; so in practice everyone does GetProcAddress
  * **Graydon**: there's gotta be a vtable in there somewhere, right? what happens if you call it & it's not there
  * **Patrick**: well, it's null and you go to some fallback
  * **Patrick**: OpenGL has a protocol for querying for features
  * **Patrick**: don't know if we rely on that exclusively at Mozilla; may be situation where few drivers implement full spec
  * **Graydon**: sure
  * **Graydon**: what I want to know: surely not *every* symbol, right? just extension?
  * **Patrick**: no, I mean *every* symbol
  * **Graydon**: what fallback can you possibly do if something basic isn't there?
  * **Patrick**: different ways of doing anything
  * **Graydon**: there must be *some* stuff that has to be there... do we seriously go case by case?
  * **Patrick**: yes. there's a symbol called s_gl_library, it's a giant vtable of every OpenGL function
  * **Graydon**: so we shim everything
  * **Patrick**: yeah
  * **Dave**: haven't we implemented all of OpenGL in a slow, portable way then?
  * **Patrick**: no, you don't polyfill, you just pick a different path
  * **Dave**: ultimately there's a fallback that says "sorry you just don't have GL"
  * **Patrick**: at that point you use different layer lib instead of GL layers
  * **Patrick**: another layer of defense: blacklist driver versions
  * **Graydon**: ok, all cool, just background... don't mind dynamic loading at all. well I hate it but we're stuck with it
  * **Graydon**: can we not expose dlopen, dlsym, etc? we're gonna need it for crate loading
  * **Graydon**: only diff between crates and C libs is different name mangling
  * **Brian**: can get a function pointer but can't call it from Rust. gotta have a wrapper that switches stacks
  * **Patrick**: those are statically generated
  * **Patrick**: should be fixed when we do extern function reform (on 0.5 schedule). should be type-based
  * **Graydon**: yeah, just provide type, and you're on your own if you gave the wrong type
  * **Patrick**: and null checks
  * **Graydon**: or conditional handler
  * **Patrick**: handler would be ideal, can return null or a polyfill
  * **Graydon**: is extern reform on anyone's list for 0.5?
  * **Patrick**: it's on the 0.5 list
  * **Graydon**: nobody picked it though
  * **Patrick**: don't think so. think niko is particularly interested
  * **Graydon**: he's doing function types, important for regions
  * **Brian**: important for servo
  * **Graydon**: if you wanna spend a servo day on it, great. just scheduling question. do we have design work to do, or just scheduling question?
  * **Brian**: purpose was just b/c bjz really wants it. making sure everyone's aware we need it
  * **Dave**: do we design needs or is it pretty straightforward?
  * **Patrick**: pretty straightforward
  * **Brian**: design sketched out on issue tracker
  * **Niko**: working towards fn reform; some obstacles to clear out of the way
  * **Graydon**: that'd be good. fn pointers are something we're gonna have to figure out. only way we can move stuff out of compiler at any point is a way of doing that

## Condition handling and item macros

  * **Graydon**: reasonably comfortable with design landing; doesn't involve running anything in destructors
  * **Graydon**: couple bugs left on it, one to do with const, one to do with item macros not existing
  * **Graydon**: posted on mailing list: boilerplate associated with this: establishing tls key, creating mostly empty struct for purpose of type dispatch. don't care if we do that extensively in this release cycle. if this bothers you I can add item macros
  * **Patrick**: aren't item macros going to be covered by john clements?
  * **Dave**: yes, and this doesn't sound too big of a deal
  * **Graydon**: just a question of implementation priorities. anyone care about item macros?
  * **Brian**: no

## Automation update

  * **Graydon**: didn't switch to buildbot yet. it is running but I'm gonna have to reconfigure it when I learn about the Amazon private cloud we're connecting to
  * **Graydon**: will require some adjustment to start using the cloud services properly, which are currently pending attention from netops. I'm in their queue
  * **Brian**: what platforms do you have hooked up to buildbot?
  * **Graydon**: everything except FreeBSD
  * **Brian**: you have a windows bot setup? did you get more windows machines?
  * **Graydon**: no, same windows machines now. but amazon will have windows
  * **Brian**: you're not using mac1 and mac2?
  * **Graydon**: yes, I'm currently using them. mac3 and mac4 I don't think I have credentials for
  * **Brian**: they don't have names you can access, probably
  * **Graydon**: that would be why!
  * **Graydon**: buildbot config is straightforward, I'll send instructions to anyone
  * **Brian**: will help whenever needed

## Multiple trailing do-blocks

  * **Graydon**: email thread on condition system: multiple trailing do-blocks
  * **Graydon**: would sometimes be useful, to pass two closures, before-after, parallel-maps, etc
  * **Brian**: a lot of cases where I wish I could keep doing blocks, one after another. also seems like a YAGNI thing. more complexity you don't necessarily need
  * **Graydon**: don't strictly need it
  * **Brian**: I'm a little wary
  * **Graydon**: okay

## Unwinding

  * **Brian**: still a priority; we've tossed around many designs; all not entirely satisfactory. one we can commit to and start working on?
  * **Brian**: one I like most: using return value that we aren't using as a status code saying we need to do unwind path. every return sets it, every call checks it
  * **Dave**: that sounds expensive
  * **Patrick**: couple mitigating factors. first of all, like two instructions on each call, not quite that bad
  * **Patrick**: LLVM has a prune-unused-arguments pass. don't know if it has a prune-unused-return-value pass. wouldn't be surprised if it already has, wouldn't be hard to add
  * **Patrick**: when we do static linking or IPO, or private functions, we can strip out checks
  * **Dave**: also inlining, right?
  * **Patrick**: yeah
  * **Patrick**: will eventually be using return value b/c of ackermann benchmark. shouldn't be too much of a problem with this scenario: if you return a struct in LLVM by value, it goes to multiple registers. that's not too bad. I think C ABI already requires this for complex numbers
  * **Brian**: if we had some system where we could identify calls that don't fail, it's basically C
  * **Patrick**: yeah, basically mirroring C as long as you can prune paths that don't fail
  * **Dave**: just don't have a sense of how common that is
  * **Brian**: we need to commit to knowing what things can fail
  * **Brian**: always kind of assumed you could fail at any point. more and more that seems not tenable. if we can define more clearly where failure happens, we can start figuring out which functions can and can't fail
  * **Patrick**: I think LLVM is a more prudent place to do this
  * **Brian**: really?
  * **Patrick**: LLVM already has all this stuff: can-unwind, cannot-unwind, etc
  * **Dave**: just concerned it's a conservative analysis, could poison everything
  * **Patrick**: but that happens even in C, just manually
  * **Dave**: true
  * **Graydon**: I'm just wondering why you're considering this better than the other models
  * **Graydon**: what about shadow stack, for example, with longjmp
  * **Patrick**: first time I've heard of this model
  * **Graydon**: windows style, like SEH
  * **Patrick**: oh
  * **Graydon**: just wondering why you've come to this, maybe I've missed conversations
  * **Patrick**: I don't know how that would work. would have to spill registers at every call?
  * **Graydon**: know, just need to write stuff that needs cleanups
  * **Patrick**: spill those registers, you mean?
  * **Graydon**: yeah
  * **Patrick**: that's basically setjmp/longjmp style. we could enable the sj/lj style in LLVM for that
  * **Graydon**: right
  * **Patrick**: would probably be more perf overhead than propagating return value. have to store frameptr, stackptr, spill bunch of registers
  * **Graydon**: why frameptr & stackptr?
  * **Patrick**: has to be part of exception handler. otherwise failing functions don't know where to jump to
  * **Graydon**: think you just need frameptr. not trying to force one decision, just thought there was an angle of integrating with platform exceptions. like native exceptions in Windows. but certainly most C code does return values
  * **Patrick**: on x64, from what I've read, Windows SEH completely redesigned: zero-cost like DWARF. unlike x86 ABI though.
  * **Graydon**: yeah they did something magical again
  * **Patrick**: dunno if LLVM has any support for this. feel like we might spend months on this
  * **Brian**: one reason for not using LLVM exception handling is fast-ISEL
  * **Patrick**: potentially could fix *that* but that might be a lot of work
  * **Patrick**: may want return values just for that reason
  * **Brian**: multiple unwinding methods is scary
  * **Graydon**: seriously not opposed to your conclusion
  * **Patrick**: no it's a good discussion to have
  * **Patrick**: there hasn't really been that much discussion
  * **Graydon**: what's weird: maybe I'm being pedantic; somewhere I read a clear description of formal difference between SEH and sj/lj; there was something technically important, but I don't remember what it was
  * **Patrick**: don't know. almost same thing, it seems, except kernel is aware of SEH to some degree, whereas sj/lj is purely userspace mechanism
  * **Patrick**: have to have a little SEH support at least in runtime, but hopefully we can just compile that with msvc. when you segfault on windows, kernel throws SEH
  * **Patrick**: or if you divide by zero or things like that
  * **Graydon**: really nice thing, honestly, about brian's proposal is it's probably most compiler-agnostic thing we can come up with. super portable. can be called from everyone, can call everyone, can probably map every other error-handling system into and out of that
  * **Graydon**: that's a pretty winning argument
  * **Patrick**: we already have people asking for msvc
  * **Graydon**: not just msvc; at some point clang will have its own linker and we'll go through that
  * **Graydon**: pretty nice to go through any DLL's
  * **Graydon**: windows have done this: in WinRT they gave up on any error-handling system, just return codes
  * **Brian**: shocking!
  * **Graydon**: they wanted to go language-agnostic, and that was the only thing that could map into multiple languages
  * **Graydon**: so, probably path of least misery
  * **Patrick**: yeah, simplest thing to get working now
  * **Patrick**: has a well-understood perf model (basically, C)
  * **Graydon**: does give rise to one issue: not perfect ability to consolidate cleanup blocks. if you have 25 function calls in a function, all cleanups call free on single pointer; then cleanup ret everywhere doesn't get particularly well consolidated by LLVM, I think. we have some stuff for consolidating landing pads, but may not be good
  * **Brian**: it's not good
  * **Patrick**: I wonder if some extension of GVM could handle this
  * **Graydon**: or we could figure out how to do this ourselves if all our cleanups are call/return. this is a not particularly pretty part of trans.rs
  * **Graydon**: don't wanna discuss forever. won't say stop, but if you wanna try to figure out exactly what SEH means and what sj/lj means, and document that somewhere, that'd help
  * **Graydon**: I can spend a day wiki-ing that up
  * **Patrick**: that would be a good thing to do
  * **Brian**: have we ever taken this to the mailing list?
  * **Brian**: lot more people around today than earlier! one may know something we don't
  * **Graydon**: I'd like to get a wiki page sketched out first, then we can ask for contributions/corrections
  * **Graydon**: but I'd definitely like to get unwinding solved this cycle
  * **Graydon**: are you starting on this?
  * **Brian**: no but I'd like to eventually
  * **Graydon**: sure, go for it & see how painful it is on a branch
  * **Graydon**: weird thing: there's a -z flag in compiler that turns off landing pads, or claims to
  * **Brian**: dunno what that's for
  * **Patrick**: mostly for back-of-envelope calculations. like what would we save in world w/o landing pads
  * **Graydon**: yeah, and didn't save much, weird...
  * **Patrick**: probably broken. I remember observing significant savings last time I looked (several months ago) so it probably just broke
  * **Graydon**: so yeah go ahead and start and I'll work on convincing myself it's the best approach
  * **Graydon**: puts last nail in coffin of everyone wanting resumable questions

## Condition system

  * **Dave**: I missed what that was about. failure data or result monad?
  * **Graydon**: mostly getting rid of result monad. essentially I'm just asking people to have a degree of faith; just sort of a gamble
  * **Graydon**: not related to unwinding. doesn't affect unwinding
  * **Graydon**: just for conveying to error site how you would like it to keep going
  * **Graydon**: in caller environment, a way to continue from error
  * **Dave**: just better syntax for result monad?
  * **Graydon**: no.
  * **Graydon**: basically a TLS key for handling policy so that code can respond to conditions in the context where they happen
  * **Graydon**: the problem with exception handling is the point was you wanted to resume, but you lost all that context where the error happened and have to start over
  * **Graydon**: you may want to abort, but you may also want to resume, and this gives you the ability to decide
  * **Dave**: this all sounds good to me
  * **Dave**: does this replace result monad?
  * **Graydon**: in many cases, yes, but we'll keep the type around for cases where people want to use the sum-type style
  * **Dave**: will there be a dynamic-scoping way to say "use this value during this computation, then revert"
  * **Graydon**: yes, that's exactly the style
  * **Dave**: awesome
  * **Brian**: one way this could be nice for actors. I'm reading about Akka. I'm pretty convinced they're the model to copy for actors at the moment
  * **Brian**: one thing they do is, they have strict hierarchy of actors, all parents responsible for failure of children. when child fails, it freezes and bounces back to parent to handle. parent executes callback: your child is about to fail: let it fail and resume? continue with next message? etc
  * **Brian**: this kinda sounds like that
  * **Graydon**: you could certainly do it across task boundaries. you'll be blocked in condition handler till returns
  * **Brian**: sure, just pointing out this gels pretty nicely with Akka's model
  * **Graydon**: common lisp people are really into the ability to drop into debugger and handle the error. synchronous aspect is interesting. fact that it blocks and can wait quite a while is totally true. it's a synchronous call
