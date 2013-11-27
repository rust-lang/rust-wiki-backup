# Agenda 11/26/2013
- Result api redesign (acrichto) https://github.com/mozilla/rust/pull/10364
- thread_local - https://github.com/mozilla/rust/pull/10312
- "enum mod" (acrichto) https://github.com/mozilla/rust/issues/10090
- Remove io_error (acrichto) https://github.com/mozilla/rust/pull/10449
- libcxxabi/libunwind to drop libstdc++ (acrichto)
- Reducing the size of NodeId, CrateNum, Name and Mrk https://github.com/mozilla/rust/pull/10670
- More strict doc comment syntax (https://github.com/mozilla/rust/issues/10638)
- rustpkg (maybe wait for Brson)
- GC, stack scanning (pnkfelix)

# Status (?)
- pnkfelix: GC, preparing Codemesh talk
- acrichto: rewriting channels, polishing off static linking, I/O bugs, chase-lev deque.
- pcwalton: closure reform, "new"

## LLVM compile speed when bootstrapping
- pnkfelix: That's on #rust-internals again
- dherman: Can we turn off some optimizations in early phases and just optimize later?
- pnkfelix: nah, just a slower compiler in later bootstraps
- acrichto: Can generate stage1 quickly, but then stage2 will be really slow
- jack: We tried this at some point. Maybe in tests?
- dherman: Maybe just get to where we're not bootstrapping?
- pnkfelix: People are judging rust as a whole based on how long it takes to build rust.
- dherman: We need to help users not feel the pain of bootstrapping.
- acrichto: Binary nightlies would help.
- larsberg: Servo would consume those as wel.
- pcwalton: Been working on those times for ~1 year on the side. Not going to get 10x better, so we need a better model and offer nightlies. We're faster than clang & gcc.
- jack: Most of the custom build logic uses single input file to .o file (gyp has this limitation). Can't have a bunch of dependencies that output one library. But we compile the whole crate at a time. Can we do one rust file to one .o file and just link?
- acrichto: Each .o file would have to have a huge metadata section. Not even sure how we'd solve cyclic dependencies. There's a bug in incr. compilation to do a chunk-at-a-time. There's nothing that blocks it; it's just a lot of work.
- pcwalton: Biggest thing would be to push on the LLVM patch that allows parallel translate & codegen. Firefox is throwing away incremental recompilation. Moveing to rust model where they concat all the .c files & builds faster. Usually if you're doing anything interesting (e.g., changing a .h), you basically have to toss & rebuild everything anyway. So incremental recompilation is probably not that interesting. Would rather get better at being parallel per-crate.
- dherman: Is build time an issue once we don't care about bootstrap? Is the servo-specific part a problem?
- pcwalton: Small complaints, but mainly problems with rustc build times.
- dherman: Probably not the best use of resources, then.
- pcwalton: The LLVM patch (from the LLVM team) is the only thing that will help.
- dherman: So, agree that nightlies would be a good solution?
- all: <thumbs up>
- dherman: RelEng setup.
- pcwalton: brson is doing releng right now. Doing ARM / android automation (high-pri for Samsung). NExt thing would be for him to build nightlies.
- jack: Were supposed to get releng help, but with their recent departure, unlikely...
- azita: Were going to commit to help us; will follow up.
- pcwalton: Without RelEng help, one engineer is full-time on build/releng stuff, per project (Rust/Servo). I include the build/bors/etc.
- dherman: We'll try to help. 

## thread local
- acrichto: there's a PR for thread local on static mut. LLVM will then flag it appropriately. It allows access to the data to be a single load instruction. Do we want to add this to the compiler? Kinda nice in the runtime so we don't have to have code lying around. But, it adds this to all code. And there's no concept of system thread in the system. In 1:1 scheduling it'd be nice, but then code for it won't work on M:N. 
- jack: What does it mean on M:N?
- acrichto: Nothing. Totally unsafe; no guarantees; etc.
- pcwalton: Our thread API should be as much like pthreads as possible so you can write programs to work on 1:1 and M:N. So, the thread-local attribute could be "task-local" instead in M:N. There could then be an OS-thread-local one that's a different thing.
- acrichto: So we'd need to swap out globals on a context switch or we'd have to codgen differently.
- pcwalton: Put addresses in the crate map and swap them out.
- acrichto: On a context switch?
- pcwalton: Different codegen.
- acrichto: I wouldn't want to accept this patch until we have a serious system for M:N and 1:1, then.
- nmatsakis: This is a big discussion and we're having it about a tiny facet of the implementation. 
- pnkfelix: How do you currently toggle between 1:1 and M:N?
- acrichto: Doesn't exist yet today. I'm not sure having a compile-time switch is a good idea. You may want to start a 1:1 scheduler on a remote thread, and might need synchronization with it... we need to work on the use cases here for the system.
- pcwalton: We need to figure out our threading model. Is it 1:1, M:N, toggle-able, or can both schedulers co-exist? No matter, I'm OK with this patch as long as the feature is unsafe.
- acrichto: It's attached to static mut, so already unsafe. But are we going to basically just do that for every LLVM feature?
- pcwalton: I see no reason not to.
- pnkfelix: Only the ones we want to support going forward...
- pcwalton: If we're going to be a systems language, people are going to want to have access to them, at least through unsafe code.
- acrichto: A feature gate would make me feel better.
- jack: How about a library compiled for 1:1 and link with M:N scheduling? Will there be two versions of all libraries?
- acrichto: I don't think it's a compile-time switch. Or, if it is, it's a switch on the executable. All libraries should work in both settings. Channels i'm rewriting to work seamlessly. Write code once and it works everywhere. There should not be multiple versions of the library.
- pcwalton: What about TLS?
- acrichto: Right now, we don't have it. Just task-local. There will still be tasks in 1:1. You can still use task-local, though admittedly that's more expensive.
- pcwalton: So in 1:1, not abstracting over thread. Still use thread APIs. What does it mean to make a task in 1:1?
- acrichto: Use task API. 
- nmatsakis: Can you always spawn a thread and a task in the API?
- acrichto: No. There will be "spawn". In 1:1, that's a thread. in M:N, that's a task.
- nmatsakis: Can we have threads and tasks exist explicitly? There are advantages to leveraging the OS infrastructure for prioritization, etc. Then you can have thread pools with separate priority levels and use the system services to balance work. That flexibility requires exposing task pools as a concept.
- pcwalton: Would be nice.
- acrichto: Different pools of M:N schedulers? You can do that today because you can have multiple pools of schedulers running around. You can set it up manually and hit "go". We have bindings to the OS thread API; it's just not the sanctioned way to do stuff.
- nmatsakis: Make sense to have something like a thread pool interface? And one is a 1:1 thread pool, one is M:N?
... <larsberg's connection crashed>
- nmatsakis: brson and I talked about this a bit.
- acrichto: Everything we use in the standard library is available, just not by default. The question is: what does spawn do? Specialized use cases might make sense.
- nmatsakis: do_spawn might use a dynamic variable to choose what to do.
- pcwalton: Interesting problem is how to get TLS to work as efficiently in 1:1 mode as it does natively?
- nmatsakis: Then expose threads and let people write code that works like that? But maybe that's inevitable...
- acrichto: If there's an OS TLS slot that holds a task object, then we can make it work. One lookup for the task, then redirect through the table to the value. Two reads instead of one. Need some compiler help.
- pcwalton: Then we could have task-local, and it'd be within a read of thread-local today.
- acrichto: Task-local today is a linear lookup in a vector. OS TLS is literally one load instruction. If we wanted safe OS-like task-local data, I think we can do it with 2-3 loads.
- nmatsakis: So rustc emits a constant like the max number of slots used in the library? How would it interact with dynamic loading? Then we'd have to resize all the TLS tables.
- acrichto: If you link to it as part of the compiler step, it'd be fine. It'll require a bit of compiler hacking, though. The thread-local attribute can be the LLVM one, and we could have a task-local one that's actually safe.
- nmatsakis: Sounds good to me.

## Mutexes
- pcwalton: Mutexes that work with 1:1 and M:N?
- acrichto: In the rust model, mutexs should not be necessary, but you shouldn't be using them. We can build one on channels, but what kind of sharing do you need?
- pcwalton: Maybe you just need different locking mechanisms for 1:1 scheduling vs. M:N scheduling.
- acrichto: You should not be able to take your locking primitives and just jam them into wildly different threading models. But maybe we need to think more about potential use cases here.
- pcwalton: So it's OK if an OS Mutex just works in 1:1, but a task mutex would get a different implementation depending on the scheduler you're running under?
- nmatsakis: OS ones are always fine if you don't block or yield, but I assume that a MutexArc is something that must have some use cases, right? Why do people use them in M:N?
- acrichto: Sure, but they're different  models.
- acrichto: Intermingling may not be something we want to support well except through channels. The only way to block an OS thread is using a mutex. If a green thread grabs a raw OS mutex, that will cause the scheduler problems.
- larsberg: May not be able to punt on mixed scheudlers. Have data-parallel threads within 1:1.
- nmatsakis: How much communication between DP and 1:1 threads?
- larsberg: Not much; DP usually communicates a result except when caching intermediate computations. 
- nmatsakis: Throughput for DP threads vs. fairness on communciations?
- larsberg: Can do some scheduling work here - e.g., DP threads don't give up timeslices, can prioritize based on it.
- acrichto: Not sure I understand what we need for mutexes here. Should we be doing this case-by-case the design for this stuff?
- nmatsakis: Worried we'll either be too simple or way over-design it.
- pcwalton: Most concered with getting 1:1 mode as efficient as possible. Then you can drop the M:N and still interoperate peacefully with other rust code. Would prefer to keep Rust code interoperable between scheduling models. So we should support will with perf-minded people with certain design styles and make sure that those still can run in M:N systems.

## GC
- pnkfelix: Niko and I have been chatting about GC. Current plan is a conservative stack scan. We may be able to use the LLVM APIs for precise stack scanning, but I'm hesitent to commit to that. I'd like to get conservative with a mostly-copying collector and maybe look at it later. Important to bring up now because certain conservative stack scan work will require changes in places we might not expect. Owned objects on the exchange heap that are not sendable can have references to stuff in the GC heap, and pointers to those objects are just a pointer that's been malloc'd. No metadata on layout, size, etc. so you can't scan it. I have a plan to deal with it - some compile-time dispatching on different objects based on whether the owned objects have any GC-allocated objects. Will probably just store the typeinfo if it has GC objects. But that will require some big changes. But, does everyone agree that we should commit to a conservative scan first?
- nmatsakis: I'm uncomfortable with being dismissive of precise scanning. The question is: are we always conservative, or is this temporary?
- pnkfelix: I'd like to do a precise scan of the exchange heap. If we have the metadata for that, then we can do precise stack scanning later. But conservative of exchange heap would mean we have no chance of being precise.
- pcwalton: I agree.
- acrichto: The contents a pointer points to will no longer be just a struct, then? A pointer + some types?
- nmatsakis: Breaks interior pointers. Putting info before the pointer, but a conservative scan might have a pointer to the middle of one. So you still need separate layout info for those stack pointers to get to the headers. I'll just keep separate metadata for that resolution.
- pcwalton: Boehm GC's two-level hash?
- pnkfelix: I'll look through it.
- nmatsakis: Possible for LLVM to have a pointer that is "four lower than the actual value"? 
- pcwalton: Don't think so. Conservative GCs work.
- pnkfelix: In the GC handbook, for Java, you want to have pointers to the start of the array <lots of details>. In any case, pointers outside the bounds of the object can arise in certain kinds of JITs.
- pcwalton: You can point just beyond the end of an array in C, but that's it.
- pnkfelix: I'll send out info on it. I suspect that if LLVM really wants to support precise GC, be careful.
- pcwalton: They're getting pretty crazy with the LLVM infra because they're going to use it for JS.
- nmatsakis: Conservative should be nice. Precise in the long-term. Maybe rely on the LLVM stuff when we can.
- pcwalton: But the gcroot from LLVM is a dead-end. Rather stick with conservative until stack_patch_points(?) or something lands.
- nmatsakis: Maybe do whatever LLVM recommends.
- larsberg: Do we know anyone who's made LLVM GC work? Corpses of grad students here.
- nmatsakis: I've heard such a person exists! But it was 2nd hand.

## reducing size of NodeId, CrateNum, Name and Mrk
- pcwalton: 32 bits is plenty

## strict doc comments
- acrichto: //// is not, but /// is? Done.

## result API redesign
- acrichto: is .ok() returning an Option OK? Any objectsion?
- jack: Why is this different than the Option API? Instead of doing .ok doesn't do an assert, right, just returns none. On Option, you just do .get.
- acrichto: Have .unwrap.
- jack: gets rid of is_ok?
- acrichto: No, gets rid of the composability apis on result and says, "just use the stuff on option instead."
- jack: Is there an example?
- pnkfelix: Something in the mailing list?
- acrichto: Maybe bring it up next meeting.
