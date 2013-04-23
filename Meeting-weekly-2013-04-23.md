# Agenda

* core -> std reorg
* valgrind
* removing existing uv bindings
* modes

# Attending

azita, brson, pcwalton, jclements, jack, niko, graydon, jld, tjc, felix

# core -> std reorg

- B: std has been kind of languishing lately
- B: nothing in there is all that captivating
- B: we have this package manager coming online
- B: pcwalton and I would like to push std out into the "ecosystem"
- B: and then rename core to std
- G: I'm generally ok with this, we discussed it some during the work week meeting
- G: I would like the "standard library" to be what we would want to standardize officially, which seems to be core
- G: the arguments for the split also had to do with binary distribution size, but I realize that those people who are concerned with size will probably be using static linking anyhow
- P: I had a bootstrapping proposal which is to rename std to ext and then slowly migrate things out of it and into rust-pkg
- P: ext has the "unofficial" connotation
- G: people often use `contrib`
- G: I think this is something where the entire conversation rests on the strength of rustpkg
- G: a good idea in principle but then we're not quite ready
- G: seems a bit premaure to assign it responsibilities until we know that rustpkg is easy to work with
- B: as an interim step, though, we do agree that the name of `core` should be `std`, right?
- B: in which case we do need to rename `std` to something
- G: sure, sure. I don't know, let's poll the mailing list. 

# valgrind

- G: had long-running conversations with LLVM+valgrind folk
- G: this is a complex topic for them to investigate
- G: no solution forthcoming, but it is one of the main thing I've been doing this week
- G: trying to isolate what the problem is and consensus on how to solve it
- G: so, strcat pointed out that we could simply zero the memory again
- G: I think that ceasing to zero memory in the place where we used to zero it is what made this visible
- G: that is arguably another way of papering over the problem
- G: but it doesn't require that we continue to write assertions
- G: we could also just disable "Branch on undefined memory"
- P: yes, that would not rergress perf
- B: but it is a big class of errors that valgrind would not catch
- jld: do we have a small test for this?
- P: there is one in upstream LLVM
- G: if you can give me another week, we can resolve this. One problem is that docs are generated based on master which is not advancing right now, but we could fix that
- G: if you are held up by master being on fire... let me know
- B: I am being held up by not being able to use valgrind
- B: I get 100s of errors and don't want to sort through them
- G: problem is that people who have the expertise to fix this are not available
- pnkfelix: can we suppress the optimization in LLVM?
- G: hard to do, it is simplify-cfg, which runs ubiquitously
- G: I'm having the same problems with the GC, false positives all the time
- G: LLVM people suggested switching to AddressSanitizer, but that doesn't detect uninit memory
- B: Do we have the ability to suppress?
- G: Sure, suppress `cond` everywhere
- G: If you're feeling really hobbled by this, then sure, we can suppress it and keep it as a high pri bug to turn it back on

# Removing existing libuv bindings

- P: So the existing uv bindings and associated infrastructure are suboptimal
- P: performance is slow
- P: frequent deadlocks
- B: also filled with unsafe code that is hard to maintain
- G: it was written for an I/O design that we have decided to move away from, partly as a result of writing that code in the first place
- P: we're working on the new bindings
- P: one of the features is that it has a raw, blocking mode
- P: people should not reach for it initially, but it basically just wraps the raw socket APIs
- P: it occurred to me that once this is up and running it's probably nicer than the existing UV bindings
- P: it has obvious problems (blocking other things that run on the scheduler) but it won't deadlock and probably performs reasonably well
- P: also the APIs will be the same so when those are ready we can transparently switch
- G: it has the same API? which UV API is this?
- P: I'm talking about high-level TCP bindings
- P: most people who write TCP apps will use rt::io::Read/Write/Stream traits
- P: and the connection APIs (not fully fleshed out)
- P: and there will be mutiple impls, one of which is raw POSIX
- P: it seems to me that the raw POSIX stuff is prob. more useful than the existing libuv stuff in practice
- G: and a lot of the people who are experimenting are using linux where thread-per-task scales
- P: then once the new stuff is ready to go people can change over
- G: Brian didn't you want to register this inside of a dynamically created factory at startup?
- G: then users shouldn't even notice because the factories will just be handing out different implementations
- P: that's the idea
- G: this is also a good test for the idea, because that is an important goal for portability
- B: one thing that's missing is timer
- B: and I imagine this will impact servo
- P: the solution for timer is probably just spawn a thread and sleep
- P: if you're sleeping more than a second or so that should be adequate

# Modes

- P: I am currently working on patch that removes modes entirely
- *general joyousness and good cheer*
- P: it seems to be mostly working so I'm hoping to have it reviewed today
- G: giant kudos to Alex who landed the patch for rustc


