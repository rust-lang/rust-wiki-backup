# Agenda 02/04/2014

* libprim (acrichto/brson) https://github.com/mozilla/rust/pull/11968
* Foo { x, y, z } (acrichto) https://github.com/mozilla/rust/pull/11994
* finally!() and std::finally (acrichto) https://github.com/mozilla/rust/pull/11905
* private std::util + reexports (acrichto) https://github.com/mozilla/rust/pull/11956
* Is using system LLVM req for 1.0 (pnkfelix) https://github.com/mozilla/rust/issues/4259
* closure rules (nmatsakis)
* operator overloading (nmatsakis)

# Attending

- azita, acrichto, brson, larsberg, nrc, pnkfelix

# Status

* pnkfelix: nil (PJS)
* acrichto: landing mutexes/io_error, next io_clone, channels, compiler flags
* brson: roadmap, diagnostics, docs, installer
* nrc: learning Rust, burn down bugs
* nmatsakis: #6801 (but largely nil)

## libprim

- brson: Wanted to have fun last week! So looked into how we could separate libstd into multiple libraries so some can be used w/o the full runtime. Did that, made this PR, and there's some contention. This PR creates two libraries out of libstd, one with no runtime dependencies at all. Really an experiment. The question is: is this a bad path/strategy? May lead to lots and lots of traits. 
- acrichto: Another idea is to use more configuration flags to produce multiple versions of libstd.
- nmatsakis: Could imagine a different type of configuration flags that supported that. Could also pull out libprim and use configuration flags with libstd. But in general, I don't like config flags.
- dherman: Eliminating modes is a good idea. They fork what code can be used with which part, and it makes it harder to reuse code.
- brson: At the low levels, people want to do anything that's possible to do. So, if you can link to libstd, you're fine, because you don't need anything. But there's an explosion of options people want...
- dherman: which?
- brson: Removal of morestack header. A few `-z` flags that flip options that make things not compatible. e.g., turning off landing pads.
- dherman: We need to figure out what we want to accomplish and make some hard calls. Pick the set of constraints and then pick a solution that satsifies those constraints. If we need to provide every possible thing and that's the main constraint, then we can't be modeless. Possibly I'm the only one saying modelessness matters. But as soon as you add modes, you have to figure out if there's a 90% case and say "other mode is only for a niche" and we avoid having people in the others? Or do are these modes 50/50% and we really have to teach people about how to document whether their libraries are compatible with one mode or another. The latter seems worse. But the worst of all is a whole bunch of modes and pure chaose because people won't know how to document their assumptions and it'll break reuse. So, if we're doing these modes, is it 90/10? Or some other setup?
- acrichto: We have binary modes, but not really source-level modes. You can take any source and recompile it anywhere. It's just that the same binary can't go everywhere.
- dherman: Still an issue for shared libraries. But if cargo usually ships source, maybe it's not a big deal.
- acrichto: Have to ship a statically-linked executable anyway. We don't have a stable ABI, etc. This would admittely make that even worse, but maybe it's just not that much of a worry. I agree that source-level modes are terrible. But maybe binary modes aren't that bad.
- dherman: Probably still want a good answer for what the defaults are that are the most compatible with embedding a library that you build to use in other systems. 
- pnkfelix: Do we still have hashes in filenames? Should the modes be in that hash?
- brson: The modes are sort of low-level things, mainly for people who can't use the standard library.
- nmatsakis: Do we really need to remove type descriptors from vtables, landing pads, etc., or are we micro-optimizing? Not convinced we need to do all this fragmentation for hypothetical targets.
- pnkfelix: Or were these switches just to enable measuring?
- nmatsakis: I'm fine with gathering data, but that might say we just need to optimize.
- acrichto: If you're compiling kernel code, you don't have an unwinding runtime, so landing pads make no sense. Same thing with morestack header. Kernel code would have to set up segment selectors, which is hard if you're writing a Rust module for the Linux kernel because you don't have control over the segment selectors. So possibly you're running in environments that don't have the infrastructure you need.
- nmatsakis: Any examples other than kernel mode?
- acrichto: Probably not.
- dherman: Part of focus is choosing what we want to do now vs. defer. Should we just consider how we can make kernel-mode work in 1.0?
- brson: Discussion has drifted to "should the compiler change its codegen" from library splitting. Those flags are really not committing to much.
- nmatsakis: It's all related to that PR, though. I'm not opposed to the kernel case. It fits into dherman's "10%" and we don't expect anyone to use it but kernel devs.
- brson: Just concerned that if we don't do the design of the standard library now, we'll have to create a new one later. Would not want to split the standard library over this use case.
- nmatsakis: I think brson is right about wanting to minimize the requirements of the standard library.
- acrichto: Could either re-export subsets of the library ala this PR. Alternatively, we have modes that let you compile one source file in multiple flavors. Those seem to be the two ways to solve this scenario. If you split the libraries, the documentation is interesting. If you have compilation modes, the docs need to expose all those modes. I'd also like to have a compile-time check to show that they are a superset of one another. So, disassembling into crates would guarantee supersets. But modes might be easier to understand.
- nmatsakis: What is the set of features? Failure?
- acrichto: allocation. Behavior on out-of-memory. 
- brson: In lowest-level, don't want failure to be the default error-handling strategy. Need to expose failure and have the user trigger task failure. Allocation is the first runtime requirement. But even the kernel is going to have to allocate. So need to know what needs to use failure and what doesn't need to use failure. Even in kernel environments can default that to an OOPS. Just want it to be possible to handle these failures.
- nmatsakis: Envision an allocation that returns a Result?
- acrichto: Libraries for kernels will probably not allocate; they will require their users provide the memory. Otherwise everything has to return an Option. But that is fundamentally different from user code.
- nmatsakis: Assuming allocation succeeds provides a much nicer library surface. 
- brson: I think we should continue this conversation in the issue tracker and cover other items today.

## operator overloading

- nmatsakis: We were looking at overloaded index operators. I wanted to ask: do we think it makes sense to land some intermediate improvements on the way towards a final design, or wait? Today we have an imperfect index and we want to have the fixed one for vector and DST. Should we take incremental steps? Not sure what edge cases would be wrong, but I'm pretty sure that there would be some if we tried to land it sooner rather than waiting for my full review. The overloaded index yields an lvalue (like an array index) so `x[3]` always returns a pointer to that item. Then, we have to implicitly, if you use in an rvalue context, derference it. 
- pnkfelix: as opposed to having two different kinds? one for lvalue and one for rvalue?
- nmatsakis: Yes. They should be the same thing. There's also a complication that when you are typechecking the index, you don't know if it's an lvalue or rvalue context, which is deeper than I'd thought. There's more than one lvalue context (e.g., mutable ones). That's a complication in the code we have to circumvent. But autoderef:
`x[3].inc()` where `inc` is an `& mut self` is an lvalue context, even though it's not clear.
```
x[3].inc(); // equiv to inc(&mut x[3])
fn inc(&mut self, ...)
```
-  acrichto: Doesn't index on a vector return a reference in C++? Sounds  like our hashmaps, where we have to explicitly say `&3`. It's a  departure from the syntactic sugar, but not from the semantics.
- nmatsakis: But then you can't emulate a built-in vector, unless we change how vectors work, too. There are already a lot of Traits in place; maybe a fourth isn't that bad. In any case, those are the semantics I had in mind.
- acrichto: How far off the intended path is the stuff you might want to land today?
- nmatsakis: I'm not sure... guess that's the important question! 
- pnkfelix: What PR?
- acrichto: 11977
- pnkfelix: I was just worried about the rvalue stuff for write barriers in the GC and avoiding them when you're just reading. But it may not matter.
- nmatsakis: Maybe for supporting maps that are not in-memory maps? e.g. the "double" map. You can't just use it in an rvalue-like manner.
- acrichto: bitvectors return bool, and you can't return `&bool`, so similar....
- acrichto: This patch only gives you, if you call `foo[3].bar()`, always calls the index. 
- nmatsakis: Yes, doesn't handle context-dependant work. 
- brson: Should we ask for the improvements before we land it?
- nmatsakis: Could probably handle autoref without too many changes, so long as it handles the explicit syntax case. It wouldn't get autoref correct, but we could do that later, since it's a bit more involved.
- acrichto: Sounds good to me.
- nmatsakis: I'll work with the author on that.

## LLVM

- pnkfelix: We need to decide on 1.0 goals for LLVM. Is it acceptable for us to be shipping our own fork of LLVM? That will change the priorities of backporting our patchs, since we would have to do that.
- brson: Our LLVM patches are mostly non-x86 and optimizaitons. So, it would be plausible for us to have our own fork from a release such as 3.4, which would allow x86 distros to use the system LLVM, but if you want some of our minor optimizations or ARM support, you need our fork.
- acrichto: rustc would need some changes
- brson: rely on a specific version, such as 3.4.
- acrichto: Sounds not that bad.
- pnkfelix: So we don't need those extra attributes?
- acrichto: fixed_stack_segment is gone. no_split_stack could just be a C function. We could make it work.
- brson: They should take that patch!
- acrichto: They refused to take it.
- brson: We should talk to sunfish :-)
- pnkfelix: At least one person said they couldn't get system LLVM to work.
- acrichto: Should that be our 1.0 strategy?
- brson: Would be nice to hear from packagers, though of course they'll just vote for a system LLVM.
- acrichto: We only statically link, though, so I don't see why they care. It's the same size bloat either way.
- pnkfelix: Don't they offer build from source options? That would be a problem (if they build Rust from source).
- brson: Some distros don't like you having static libraries. I think we should make the minimal effort to get on top of the release fork and make it work on top of x86. Doesn't seem like that big of a deal. But then we have to kind of commit to that forever, that rust is compatible with a release version of LLVM forever.
- acrichto: Meh. We can change how we release/build rust at any point.
- brson: Could also say it's not a goal for us but is something we would strongly like and we would really appreciate a community member helping us land the LLVM patches upstream. We could also provide our own dpkg files, alongside the binaries we make available anyway.
- pnkfelix: My instinct is that I don't want us to spend too much effort getting something working that we don't care about, but I also would like to help a community member take up this flag. I'm worried that we willy-nilly make LLVM changes to suit our needs and make it difficult to ever land anything upstream.
- brson: No more plans for LLVM features.
- acrichto: Just build-system related. We will never add a big feature to LLVM.
- brson: Let's say that's our intent. We want to be compatible with system LLVM for 1.0, but we need help to do it.
