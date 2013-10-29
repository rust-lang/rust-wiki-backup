# Agenda 10/29/2013
- segmented stacks (acrichto)
- priv keyword, #8122 (acrichto)
- rustpkg remote package testing (tjc)
- placement new (pcwalton)

# Status
- acrichto: landing #9901, organizing rt::io::file, extracting uv, 
- nmatsakis: landing multiple lifetimes, fixing mut permissiveness, dtors/temp-lifetimes
- pnkfelix: hacking on DST
- pcwalton: function reform

## segmented stacks
- acrichto: Using the morestack function from LLVM to detect stack overflow and abort. There has been discussion on the mailing list about this. The first thing I want to do is that we haven't officially said if we're going to use segmented stacks or not. There's code for it but it's all turned off. It seems the opinion is that we'll remove segmented stacks but we should make this decision.
- tjc: What are guard zones?
- a: No stack checks, you round up the page size and the runtime puts two unmapped pages at the ened, so you'd hit that if you overflowed. You'd lazily map in your entire stack up to that point and when you hit it you abort somehow.
- tjc: What are the alternatives?
- a: The thing with segmented stacks is that you want them on 32 bit because the stacks can expand as needed. What we currently do is a giant 4MB stack. THe bad parts of what we do now (it's still not clear) is some correct way to deal with stack overflow. Ideally we'd just have task failure. The biggest problem with that is that currently you'd leak the arguments to the last function frame. it's correct if you don't have any arguments. Because of that is why I decided to make it abort. Segmented stacks wouldn't fix that.
- tjc: Why would we leak the last argument frame?
- a: Right now with segmented stacks we'd abort. But you leak them because there is no landing pad for the arguments. When you throw a c++ exception and it triggers unwindows, it blows right past that. There are ways to fix that, but they are all very painful and there doesn't seem to be an elegant solution.
- brson: It gets harder if we use a guard page.
- jld: because you'd segfault on some access.
- a: The good part about unwinding at the beginning is that you know you are unwinding. It's also an LLVM opt problem if we unwind. LLVM marks functions as no-unwind and then optimizes based on that. So if we have arbitrary unwinding then the only functions.. ???
- niko: Using a guard page doesn't rule out graceful handling of errors. The usual technique is to touch the stack in the prolog. I think windows even requires this. Moreover I think we should agree that aborting is not really an option. Does anyone disagree with that? Aborting when we run out of stack seems like an ungraceful exit.
- tjc: It's failsafe behavior
- niko: the JVM, Python VM, JS VMs, etc manage to handle this situation. I think there are some tricky scenarios, but it's kind of orthogonal. Right now what we do is unwind through them. That's probably ok. You could allow unsafe code to catch and repropagate the error
- a: I agree that aborting is not really acceptable. We should be striving to someday have an unwinding solution.
- niko: If guard pages prove too challenging we could have some check in the prolog like morestack. We could just have LLVM subsitute the final stack size and have some code that loads the stack frame and compare against the desired limit. There are other options.
- jld: Does -fasync-unwind-tables exist and work in this world?
- a: With morestack?
- jld: with LLVM. The gcc option gives you extra unwind info at every instruction boundary. I don't know if anything like this exists in the LLVM world.
- a: Not sure.
- jld: I know GCC has unwind tables for ARM that are wrong if you are in the prolog or epilog. If that could work it might allow dealing with guard page segfaults coming from something in the function.
- a: This is kind of intertwined with how do we protect against stack overflow. We initially said we'd do segmented stacks. Are we agreed that a monolithic stack with overflow protection is where we're going to go from now on?
- niko: I'm sad to make that decision but I'm ready to make it.
- b: I'm ready to see segmented stacks go.
- tjc: I don't disagree.
- p: go's minimum stack size went from 4k to 8k in 1.1 because of this. I see no advantage to segmented stacks unless your minimum size is < 4k. There is no advantage. Go's bump from 4k to 8k hints that we can't really make 4k work.
- n: 64-bit is the future :-)
- a: 32bit will be in people's pockets for a while
- p: will memory mapping on linux bite us?
- a: Can probably work around it at some point.
- tjc: Felix is trying to call in. Let's not hang up on him :-)
- a: Let's draft a mailing list thing about this. Segmented stacks are gone. Also closely related: how do we protect against stack overflow. Talk about now, or move on?
- n: Have we established graceful handling vs. abort of process?
- p: Need to prevent the image decoder from taking down all of servo. We want fine-grained failure control. But stack overflow is one of the most common ways for tasks to fail.
- n: Good argument for dumping segmented stacks to prevent us from overflowing all of available memory.
- p: In ye-olde-Rust, we were going to have strict memory limits in place. Easier before the exchange heap, so that strategy might not have worked anyway.
- n: Separate and more complicated issue. We're agreed that we lose segmented stack, but gain graceful failure.
- b: 1.0?
- n: No. For now, stack overflow does what it does. Ideally not segfaulting.
- a: Now it tells you segfault and then crashes!
- n: Huge improvement from just crashing.
- jld: rustpkg/rustc, got Illegal Signal instruction. Printed it, but the parent process ate the error messages.
- a: Do we want to talk about how to do stacks now? Or move on to something else?
- a: Now that we don't have segmented stacks, how do we do monolithic ones? We do malloc(BIG_NUM) and pray it gets mapped incrementally. Use TLS and morestack to detect overflow. Strategy works but is not guaranteed to lazily map and has the prologue issue. The other option is to manually map in the stacks. Allocate the address space and mmap the result in via a fault handler. 
- jld: to avoid the limit on the amount of space we can map? Why not just mmap the stack?
- a: Have to make sure we're guaranteed it's lazily mapped in
- p: Documented as doing so...
- jld: This issue has come up elsewhere. Bugs related to madvise as an optimized mzero.
- b: Just madvise pages to be discarded. Works on all unixes
- n: Is there any Window equivalent?
- a: You can allocate and drop and reserve an address space yourself. Plenty of API functions to do what we want.
- p: There are on Win8 PrefectVirtualMemory. That's fine - Rust+Windows. Supporting old versions of Windows is not good.
- n: There will be a way to solve this problem eventually.
- b: Should we work on this right now? Before 1.0?
- a: It's working fine now. I didn't test on Windows, but on Linux and OSX, they are lazily mapped in. 8MB allocation, get 4k of usage. Be nice to figure out for 1.0. Final thing I can think of is detection of stack overflow.
- b: Stick with what we have because it works (ed.: for now).
- p: it crashes the process!
- b: out of scope for 1.0.
- a: Kind of a pain if you use rust in another environment.
- p: C is broken by not requiring more_stack, etc. 
- a: But it makes it hard for people to write a Rust module. Right now, people have to compile to LLVM bytecode and link it in like that to integrate Rust in a C program.
- p: We can make that easier for that scenario, but I don't want to go to the C model of running over the stack limit => process death.
- a: Or use guard pages and no prolog.
- p: No guarantee you will hit the guard page.
- n: Have to add a touch on entry.
- pnkfelix: We can force it to happen, right?
- p: How do we guarantee we hit the guard page? Touch on function entry?
- n: That's what compilers do if they allocate > 4k.
- p: We need it for more functions.
- jld: One byte per size of the region.
- p: "stack protector all"?
- jld: Is that a stack check or just a canary?
- p: probably a canary.
- a: Big thing about guard pages/zones, we need a prologue anyway because we can't fail at an arbitrary point within the function.
- n: If your stack frame is > 4k, it checks, but calls __checkstack(). We'd have to write our own thing that doesn't rely on magic symbols to make life easy for kernel devs.
- jld: Kernel devs shouldn't be allocating large stack frames shouldn't be a problem. They're used to keeping it small.
- a: Still have the weird LLVM bitcode or an __morestack() function definition required for them.
- n: My guard page concern is that we'll find it's technically challenging. Inserting the store is not a problem. But can we get good handling if that touch fails, do we unwind cleanly (have landing pad, etc.)? Don't want to lock into this and then find out we don't have the API capabilities.
- a: Think we can extend what we have now. The problem with guard zones is catching signals is hard. Is mmap safe inside a signal handler?
- b: Lot of work. Not necessary for right now.
- n: Leave it ill-specified for now and come back to it later.
- a: Strategy for now is pretty good. Don't change much for 1.0 because it's a lot of effort.

## rustpkg
- tjc: Had remote package testing but the tests were failing sporatically so we removed it. That's caused some regressions. I fixed one. What's the best way to go about it? Private git server?
- a: github and ask if it errors?
- p: Strongly against outbound network connections on our bots. Big problems on Firefox. zombo.com went down and lost over a day of FF development because the site went down. RelEng has now firewalled their bots.
- n: How about a local webserver? 127.0.0.1? 
- p: Intranet. example.com gets redirected to a local server.
- tjc: What are the logistics?
- n: Home machines, too.
- a: Hit github.com and ignore the failure?
- tjc: Defeats the test. The task failure used to also make it hard to determine if it was a network failure or something else. Now I can test for it, but it'd be nice to have passing tests.
- p: RelEng firewalls the infrastructure. Part of testing is no outbound requests.
- j: It's not if, but when we move to RelEng. We had to disable that stuff in Servo, too.
- b: Does git have its own simple server? Just spin it up on localhost.
- j: Git works over http. Use that.
- p: When you scale up, you can also DDOS zombo.com (or github) resulting in a ban.
- j: Will check if git can do that. Otherwise, we can set up an internal server.
- b: Don't like having build infrastructure that requires you are running on the intranet.
- tjc: More flags?

## placement new
- tjc: What was that?
- p: Takes a type that implements a smart pointer trait:
```
    let my_thing = new(RC) Thing::init("hello");
```
- p: It allocates an RC box by calling RC::alloc. Then, it calls Thing::init and sets the return pointer to the allocated space. Similar in spirit to:
```
    let my_thing = RC::new(Thing::init("hello"));
```
- tjc: new paramterized over an allocator?
- p: Yes. Similar in spirit, but does not require space on the stack for Thing::init and a move into the RC box. Maybe LLVM can optimize that out; maybe not. In general, we try to create language constructs that don't rely on LLVM optimizations that are not guaranteed.
- p: RC should be a function because, as Igor brought up, you may want something like:
```
    new(my_vector.allocate_space_at_end) Thing::init("hello")
```
- pnkfelix: A value, not function right? Might be other kinds of traits.
- p: kind of like push, except that it allows you to create it within the constructor. C++ can do this, right?
- b: We don't have first-class methods right now, right?
- p: It would have to be a call...
- n: It would have to be a value that implemented a trait, and then you could accommodate a couple of nice use cases:
```
new(arena) Foo
new(my_vector.space_at_end()) Foo
```
- p: Good point.
- n: Could have a struct thunk that implements the relevant trait.
- p: Can implement all but new as a library, then.
- n: Need to think more about what this will look like.
- tjc: When is this performance-critical?
- p: Don't know.
- tjc: Why do it?
- p: No measurements, but if we move @ into a library and will add a move and lose perf.
- n: Want smart pointers more equal, allocated through the same mechanism. Also allows unsized values, which are not possible today. Don't want to do that at first.
- tjc: How is unsized different from dynamic?
- n: Same thing. Unsized = runtime size, so a little imprecise.
- a: Is that the syntax?
- p: Really, I want to know if I can reserve ```new``` as a keyword. It will result in a lot of breakage. I recommend changing from new to init. And new_from_X to from_X.
- a: are we converging on C++?
- p: There are a few things it did right - we've been converging slowly on C++ with many features.
- pnkfelix: C++ memory address? original justification for placement new. Was really useful for taking an existing placement new and separating it into the allocation and initialization portions.
- p: Could wrap in a value that makes it work.
- n: There's a trait we need to talk about. Coudl write something like:
```
new(AtPtr(p)) expr
where AtPtr is an unsafe fn that basically just wraps a *T
impl Alloc for AtPtr {
    fn alloc() -> *() { self.ptr }
}
```
- n: Assuming we would then implement the alloc trait. Question: how do we get the type of the result?
- p: Opens the door to pattern  matching syntax for smart pointers:
```
    let new(GC) x = new(GC) 42;
```
- p: Uses new to destructure something. Some people (dherman) found this objectionable.
- pnkfelix: I can't read that.
- n: Analogous, but ick.
- pnkfelix: I vote for \nu{}!
- a: Will this be common?
- b: This will be for everything. Instead of ~foo, \nu{} foo.
- b: Nobody likes that?
- a: Changes a lot of code does not mean we shouldn't do it...
- n: I like that it reads as what it obviously does.
- a: A little in favor of this. We have ~ and @. If we throw away @, then we just have ~. And then new is just the wordy version of ~.
- p: Orthogonal and makes them extensible.
- tjc: Prevents the four different types of pointers argument.
- jld: Syntax of types?
- p: Opinion ~ to * conversion from dherman and myself. Then * is just a normal pointer.
- n: The main question is would owned pointers also have a name or be vestigal? Still special: linear values and so forth. Feels like a gratuitous change.
- a: Need pattern matching over values contained in smart pointers.
- p: Need to destructure for Okasaki-style red-black trees. At least for unique pointers. Pattern matchin over others can be saved for Rust 2.0.
- jld: Does the pattern syntax have to specify what kind of pointer?
- n: No. I was thinking what you might be thinking. We could have a dereference pattern. We currently have patterns for matching the construction styles. It's known for sure that you're dereferncing the exact kind of pointer.
- p: That would be nice. Not writing new(GC) on the LHS would be nice.
- n: That syntax is a non-starter. Just one step too far.
- p: Everything complex is weird when you destructure it.
- p: Obj-C uses alloc and init. Multiple options here.
- n: My vote if we're going to unify is that new is the obvious choice as far as preference. 
- p: The community on IRC is against these changes.
- pnkfelix: is there a way to use sugar to keep ~?
- p: Make @ a valid identifier.
- pnkfelix: ```use @=GC``` at the top? And make it work in a value and binding context?
- n: Making @ a valid identifier would be ugly...
- p: Works for TeX!
- n: It opens up the doors...
- p: Fine with allowing @ to be rebound. Feeling when we start writing in this, we'll quickly forget the pain of the change or requirement to use the @ symbol.
- pnkfelix: But some common coding patterns would be easily abstracted with just a little bit of sugar here.
- n: Do we just need @?
- pnkfelix: Multiple sigils. One for RC, one for GC, one for owned would be nice. Maybe not the right call, but avoiding the wordiness is good.
- a: Let's make a formal document / blog post, put it on the mailing list. Discuss more with the community.
- n: Not just renaming a lot of stuff; this is a big change. Don't object to it, though, but we'd better be sure. Don't want to do this and then go back.
- p: Was originally thinking Thing::Thing instead of init, but people were not happy about it.
- j: What happened to the 'in' keyword?
- n: Don't like using that to mean allocation, doesn't "say allocation" to me
- p: It reads backwards, for one thing. Also seems like not doing what C++ does for no particular reason.
- n: It doesn't unify allocating owned objects with other objects.
- p: ```Foo in;``` is just weird.
- p: Constructors usually take arguments. So just name it from_XX. Path::from_string is better than Path::new
- b: Only if it's a conversion. If you just have a bunch of parameters, from_options is kinda weird.
- p: Was using from_ all the time in my compositor code...
- a: from_cstring, from_path, etc. 
- p: Then maybe init is the right thing. Path::Path also reads nice.
- n: Don't like because rename type = rename constructor, which is a C++ annoyance. And it encourages you to shorten your type names to avoid typing long constructors.
- tjc: Over time, defer to mailing list?
- n: Need a writeup. This will be contentious no matter what we do.
