## Attending

Brian, Graydon, Jesse, Dave, Niko, Patrick, Tim, Marijn

## Argument modes

* Niko: still no agenda in advance; any issues?
* Marijn: I have some
* Patrick: I have some
* Marijn: RFC today for argument modes -- everyone agrees they're not a great thing; with mono we can get rid of by-val / by-ref distinction; something like by-copy will remain needed; saves a lot of copies from constructor functions, mostly enums; unless someone has alternative, should keep at least two modes
* Graydon: which two?
* Patrick: not sure I totally understand problem. could you explain?
* Marijn: everyone understand by-copy?
* Niko: explain anyway?
* Marijn: says "this function needs to own argument"
* Jesse: two modes are really "with ownership" and "without ownership"; caller can decide whether copying or moving
* Niko: not a big fan of keeping copy mode; creates implicit copies
* Marijn: it removes implicit copies as it currently is
* Niko: in one sense, yeah; but I'd like to think about this a little bit; not sure why we need it per se; I'd like to see how regions turn out
* Marijn: I don't think regions are relevant; say you want to call enum ctor with something big (some(vec)) -- explicitly at caller site annotate as move
* Niko: disagree; pass it by-value, which means ownership
* Marijn: by-value is by-copy essentially
* Niko: maybe we should just have by-value period; when you pass a pointer by value that's cheap
* Marijn: so a function would always own its arguments
* Niko: always, yeah
* Marijn: ampersands will be a pain
* Graydon: winds up looking like C; actually okay with that; semantics much better understood; I know there are a lot of ampersands (asterisks in C), but I think systems people are comfortable with that
* Patrick: I agree with Graydon; I'd love to eliminate by-pointer/by-value
* Niko: I think I agree with Graydon as well
* Marijn: if that works out, we can drop argument modes altogether; that would of course be very desirable; let's try
* Marijn: other thing: troubleshooting thing: pushed patch to make lots of symbols internal to our libraries; decreased speed; anyone have an idea why that might be?
* Niko: is check to decide what's internal/external expensive?
* Marijn: if I change code to only remove lines that call setVisibility then slowness goes away
* Patrick: internal vs external makes LLVM optimizations totally different; think it optimizes internal much better
* Marijn: doesn't show up in unoptimized build, so plausible
* Niko: sounds like a good patch then
* Marijn: agree
* Patrick: only for optimized builds; shouldn't treat regressions in optimized builds as seriously
* Niko: agree

## Release status

* Graydon: release status: want to try try try to get release package going; profiler I've been working on probably won't land; some spooky bugs and not critical; will try to get x64 ABI fix done; but then hoping to package by the end of the week or early next; okay?
* Niko: great
* Graydon: I did make a new profiler, in a branch in my private repo; fair amount of interesting info on where we're spending instr counts; if anyone wants to do perf hunting contact me, I'll show you how to use it

## Emerging languages 2012

* Graydon: failed to mention a while ago: we got an invite to emerging languages 2012; satellite of strange loop, in St. Louis; anyone feels like getting involved; end of september

## Dynamic scope

* Graydon: maybe better to discuss in a bug, but: we've talked a little about dynamic scope; dynamicalling scoped exception handling, variables, neither, both; particularly I've been thinking there are some experiments concerning region-based allocation, and it occurs to me dynamic stacks of arenas are not a bad candidate for dynamically-acquired variables; would be very pervasive; anyone have gut reactions?
* Marijn: one data point: our current approach to safe reference analysis would fall apart, but it's already shaky so maybe we should abandon that
* Patrick: several points: first, consensus is they're not a disaster; not opposed in principle; in terms of arenas, we'd have to think about the types to make sure that works; tricky when storing arenas in these things, b/c it imparts a hidden region polymorphism on the functions
* Niko: have to build it into the type
* Patrick: think it could work but have to be careful; other thing is that classes can impart a little bit of state without having to pass contexts around explicitly, so my instinct is we should try classes for a bit and see how it feels
* Niko: I think impls are even possibly a better fit for this than classes; can spread across multiple files
* Patrick: could use traits
* Niko: yeah
* Marijn: dynamically scoped thing can be seen through functions that don't cooperate in the passing, so it's more powerful; 4 years full-time clisp they were quite awesome
* Patrick: biggest motivation right now is trans, and I think we could experiment with classes
* Dave: invariant we have right now: no dependency on state; would lose that invariant
* Niko: could have a couple special arenas that serve this purpose; so we'd have some dynamic variables for special cases and still keep that property, though it would lose the general power
* Patrick: let's try to explore that
* Patrick: kind of like auto-release pools
* Niko: yeah
* Graydon: so, some interest there, not straight-up horror; not sure how well that dovetails with concept of at-error-site handling; unwinding is a diff issue, but maybe recover at error site potentially relates; this is all 0.3 / 0.4 stuff
* Niko: I've been considering whether we should have a monadic style, like a "result monad" b/c it's a pain to propagate errors
* Graydon: yeah, needs more design/exploration time

## Conveniences

* Niko: I had something else but I forgot; I'll email when it comes back to me; not urgent
* Dave: packing bits and pointer types, special "pack" syntax?
* Patrick: maybe just a library with accessors
* Patrick: how much do we want to be interoperable with C?
* Niko: if we go crazy optimizing, it will be hard to predict layout
* Marijn: annotation is best approach: "just leave this in simple form"
* Dave: sort of like 'extern "C"'
* Graydon: pragma of some sort
* Jesse: another Moz problem we have is -1 is a special value, everything else should be unsigned; having sentinel values but not wanting an extra word
* Dave: seems like you could have another library for that
* Graydon: enums allow discriminant values; if you could say "this enum has to fit within this rep type"
* Dave: library that's equivalent to opt(int) but size-optimized?
* Graydon: that's probably the best way to approach it, a newtyped-int whose operations are hidden; you can check for the sentinel value
* Jesse: want alt
* Niko: could have a conversion to opt(int)
* Dave: need to make sure that generates code equivalent in perf to checking for -1
* Jesse: also may want more than one sentinel value
* Niko: might be nice to have a partial-function syntax, but that's another story altogether; basically an alt syntax that is a closure; I've wanted that many times
* Jesse: me too
* Dave: these sound like nice conveniences worth RFC's or bugs

## Arenas experiment

* Patrick: do people mind if I experiment with making block contexts into an arena; trans function signatures will get unglier, will need fcx
* Niko: would rather they become impls to avoid this ugliness; but I don't mind experimenting; been wanting them to be impls anyway
* Marijn: not sure it'll feel much better; stuff becomes no less explicit than it currently is
* Niko: guess it's a matter of taste; to me it feels better, even if it's not really better
* Patrick: I think it will improve the performance though
* Niko: if you use an arena, you mean
* Patrick: biggest sources of traffic for malloc are block context, bit of hash map allocation, and also some really inefficient things resulting from copying vectors; should start moving fwd on fixed-length vectors, slices, etc for 0.3; stuff that makes them not unique and allows you to create them on the state; egregious example: io_writer::write_char allocates a big vector just to write and then frees it
* Graydon: agreed; should come up as soon as you feel confident regions are useful enough to build on
* Marijn: a great use case to try out how well they work; give it a shot with arenas for block contexts
* Graydon: cool, I'll keep pushing towards 0.2 release; try not to land destabilizing builds; exciting experiments go in branches for now

## jemalloc

* Patrick: what about jemalloc?
* Graydon: how's that going?
* Brian: I have a branch with it; basically working, kinda lost enthusiasm for it since it barely dented graphs on the bot; noticeable improvement but not as noticeable as a lot of the swings happening lately; does create another configure script that has to be run
* Jesse: what were you measuring? big advantage in Firefox was fragmentation; after allocating and freeing a lot of things, not as much free space fragmentation
* Brian: wasn't doing anything like that, just our standard benchmark
* Graydon: windows malloc has tremendous perf problems; think you probably won't see a lot of the effect it has via perf scripts
* Niko: maybe we should have an intern work on performance benchmarks
* Graydon: that could be great
* Dave: great idea
* Brian: if I land jemalloc... I'll do OSX tests and Windows and see what happens
* Graydon: not opposed
* Brian: I'll try to come up with a fragmentation test
* Niko: probably have to do allocations in weird orders
* Patrick: could port v8 splay benchmark, think that's a good test of fragmentation; creates insane representation of a tree, adds and removes nodes in crazy orders
* Brian: if I do land jemalloc, is everyone ok with another configure script, or should I integrate jemalloc's configure script into ours? means another autoconf script
* Niko: another thing I have to do manually?
* Brian: all automated, just more scripts
* Graydon: it's not gonna run often; as long as rerunning is only predicated on changes to jemalloc; we don't reconfigure libuv often, for example
* Niko: but we do run configure a lot now
* Graydon: I made it oversensitive somehow
* Niko: for me it runs every make
* Jesse: can LLVM make optimizations all the way through jemalloc?
* Niko: what does that mean?
* Jesse: if jemalloc has function with early return, a particular Rust caller's never gonna hit that early return, can jemalloc be inlined?
* Niko: no
* Patrick: if you put it in intrinsics and statically linked... since malloc is so incredibly dynamic, I have a hard time imagining LLVM gaining much from inlining any part of it
* Brian: no
* Niko: not only that, but we'd have to inline the source, b/c LLVM doesn't pull things from assembly
* Patrick: that's not that hard, but I just don't think it'll matter
* Jesse: if you have a fixed-size allocation, you don't need the function with all the branches
* Niko: yeah, but all that is is where it looks up the bucket index; that's a small part of the function
