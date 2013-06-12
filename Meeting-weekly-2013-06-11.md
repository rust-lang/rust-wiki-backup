## Agenda

- proposal for effects (bblum)
- end of master/incoming split? (graydon)
- alloc expressions (graydon)

## Attending

azita, pcwalton, eston, aaron, eric reed, bblum, brson, graydon, tim, sully, jack, niko, felix, jld, eatkinson, tkuehn

## Proposal for effects

- https://github.com/mozilla/rust/wiki/Proposal-for-effects
- bblum: want to be able to ask compiler to prevent a function from failing (or have other effects)
- bblum: hoping to achieve this with a lightweight annotation syntax that can mostly be inferred, with some limits
- bblum: one limit is that functions might need to be polymorphic
- bblum: I should lead with use cases:
  - main use case is that we can update the drop trait to indicate that the destructor should not fail, thus preventing crash when a destructor fails
- G: This can also be addressed via hard/soft unwind idea, not yet implemented
- G: Basically a version of unwind that does not run the destructors
- G: "Best effort" unwinding
- bblum: Would this free owned pointers?
- G: Yes but not run any more user code
- G: There is a recursive boundary on that, so if the dtor fails, it will re-run in soft mode, and then continue with hard mode as we proceed to unwind further
- G: Clearly just exiting the runtime is not the appropriate behavior
- bblum: in my mind, crashing runtime is sort of a dynamic assertion that checks for same thing
- G: there are a lot of reasons that one might fail which are not surfaced in the language,
- G: for example failure to allocate memory, bounds violations, etc
- G: receiving signals from the outside world
- bblum: there is a section in the proposal about actions that technically might have a certain effect but we could consider omitting them from the effect system
- bblum: for example, stack growth
- G: right but if we can't truly rely on non-failure, then we still need a way to handle dtors that fail
- bblum: it would make it so that we still need a good mechanism for handling the double fault problem
- bblum: but it would still prevent careless users from crashing their program by writing bad code in the destructor
- bblum: syntax. main reason I see an annotation burden arising are places where an effect is forbidden and you still want to write some polymorphic code and you want to use and have the effect system be able to figure out that you are using the code in a way that won't actually fail. I don't intend to do flow analysis but we have to be aware of possibly parameterizing traits over effects. In some cases you might need to write effect variables explicitly as detailed in the proposal.
- bblum: Examples:
```
#[lang="effect_fail"]
effect Fail;

trait Drop { fn finalize(self) wont(Fail); }

fn call_some_callback(blk: fn() wont(Fail)) wont(Fail)

fn foo<T: Trait<wont(Fail)>>(x: T) wont(Fail) {
    x.some_method();
}
fn foo<%e: wont(Fail), T: Trait<%e>>(x: T) effect(%e) wont(Fail) {
    x.some_method();
}

struct Foo {
    x: ~fn() wont(Fail)
}

```

- bblum: the `wont(Fail)` would be part of the function signature, part of its inferred type
- bblum: `call_some_callback` would be prevented from being invoked with a closure that itself might fail, it would also declare that `call_some_callback` itself does not fail
- bblum: you can also have more heavyweight cases with bounds and methods
- bblum: the first version of `foo` is shorthand for a second version that uses effect variables
- bblum: I've outlined in the proposal some sort of reasonable defaults, I don't know to what extend we'll be able to make use of inference. 
- bblum: If you have a data structure with a fn pointer inside (`struct Foo` - ed) you might want to parameterize the data structure with effect variables too or specify effects on the types of its fields
- bblum: Some problems I've run into. There is a question about what happens when a library author doesn't want to think about effects but the library user does. This is hard to solve. If Alice writes a library, and Bob uses it, and Bob needs it not to fail, but then Alice upgrades her library such that some function that used to be infallible can fail but now does, Bob's code might start failing to compile. 
- bblum: More technical problems are the actions where we might want to squelch the effects (assertions, stack overflow). Assertions are problematic because writing an assertion shouldn't have a failure effect, but of course they can, which means that dtors would crash, unless we have this double fault mode. Still separating out the assertions into trust-me fns seems unfortunate.
- bblum: Other examples are `debug!` (should it have the I/O effect?).

```
fn foo() wont(IO) {
    let x = @...; // trigger GC -- illegal
}
fn bar() {
    let y = @FileHandle(...);
    foo();
}
```

- bblum: Another troubling thing is when you have a destructor that might have some arbitrary effect, e.g., it performs I/O, and then you put it into a managed box. 
- bblum: You don't know when the destructor will run. For example, the function `foo()` above may cause I/O to occur by allocating, which could in turn trigger and GC and run the dtor.
- bblum: My best thought for how to deal with this is to say that garbage collection has an aggregate effect that also implies a bunch of others (but not failure).
- bblum: So that implies that banning I/O would require banning GC.
- bblum: That's basically the proposal, there are more details.
- bblum: What do you think?
- pnkfelix: I'm a little nervous myself, trying to digest
- pcwalton: I read it over, I think it's something that I have wanted for a while, but I am nervous about tying any language features (such as dtors) to this mechanism, and I'm nervous about tying a Rust 1.0 schedule to this, but it sounds like this doesn't really solve dtor failure anyway. I'd put it behind a flag until it's baked. But i'm fine with experimentation in this regard. At this point we are trying to pare down the language but that doesn't mean we shouldn't do experiments for things that might pan out in the future. Seems like there will be some cycles of "implement-experiment-refine" yet to go,particularly as there are still some unknowns. But that's what flags are for.
- N: I think I basically agree I'm just not sure where to place in the priority list. The basic design seems like roughly what we'll end up with if we pursue this path.
- jld: I'm concerned that the standard library will get frequently broken 
- N: This does rely on whole crate inference which should in theory help with this issue
- bblum: I think what jed means is that I add in annotations like "these vectors ops are not unsafe" but then somebody else makes changes, they will be able to build and push their change without having them be checked,  that is true, that's the hazard of a flag.
- G: I just don't feel like we have the budget for this. Both cognitively and workwise.
- G: There are a lot of technical issues to work through but I don't feel like this is the phase we're in right now which is mostly one of removal and our schedule is very tight.  It's an interesting idea but we've tried it in the past and found it very difficult and we're not currently doing new stuff where we launch out into new exploratory zones, we're trying to pare down
- N: I was also just thinking before you spoke "but man does this feel like a lot of new stuff"
- G: I get an e-mail every couple of days saying "when you going to stop adding stuff" so I don't really feel like we can be doing this type of new work right now
- PF: maybe right way to look at this is what we would want to do to allow this sort of thing in the future?
- N: What do you mean?
- G: Like reserve keywords?
- PF: Maybe? I guess there is syntactic stuff like the potential changes to the grammar, but I was more concerned about...well, I guess that beyond that, it could really be a separate static analysis.
- P: I think this conversation has two parts. One is: is this a language feature that's worth exploring? The other is "do we have the staff / scheduling constraints". The second strikes me as more of an internal conversation. I'm not sure we can have this conversation without Dave here. It's kind of an HR question, right?
- G: My concern is very much cognitive. When we're cutting, one of the reasons we're cutting is to pare down to the language thoughts.
- N: ... concurs ...
- P: I think it's pretty clear that this will not be a Rust 1.0 feature
- N: Won't it just bitrot away?
- Azita: Why is this something we should do in a different meeting?
- P: ... some discussion ensues ...
- G: OK, can we then redirect this discussion elsewhere and revisit? I think we've gathered enough reactions

# End of master/incoming split (graydon)

- G: We've had this split for a long time, it's confusing. However, we've had this belief that master should always be green and incoming is not green
- G: Not clear to me that master is very useful, non-building changesets still wind up in master, and it's only the 32 bit hosts that are problematic 
- brson: and free bsd
- G: Mostly a conversation between the people most on top of the build system
- brson: I think we should just drop incoming
- jld: we could have another branch that points to the last build where all tests passed
- G: "green"
- brson: I have some questions related to this. What's your timeline for adding more configurations and more bots to the auto build, so that we stop breaking master.
- G: Don't have a strong timeline for that. Saw that you are using bors for Servo.
- P: But there are many deficiencies. not having access to build logs is problematic.
- G: Trying to put as much time as I can on GC to get something landed before 0.7
- G: so if we disable incoming, that should free up some resources, I can probably do that in a similar sort of timeframe
- G: if we turn on valgrind on bors though I fear we will not be able to land things in a timely fashion
- G: I think enabling valgrind triples build time
- brson: as long as we continue to run builds on master as well, I think it's not so bad to not run valgrind on integration builds, lately it's been the cross-compile stuff that's been killing us and 32-bit issues
- G: I think we can add 32-bit stuff but valgrind is just going to be too slow
- P: We may be able to use ASAN integration
- G: with valgrind we'll be up to 2 hours per integration, not great, vs 30 mins today.
- B: If we just did the full test suite on auto, then we wouldn't have to build master at all, right?
- G: True, and if we breakup the test suite so that it runs 25% on 4 machines that run in parallel during integration, that would lower latency (1 hour -> 25 minutes)
- G: Problem is the serial bottleneck
- G: I don't really know why compile step takes 1 hour on master build, perhaps the cross targets? Anyway, let's take this off this meeting

# Alloc expressions

- G: We talked a bit last week about alloc expressions
- G: Had a kind of team sub-meeting about GC, DST, closures
- G: Discussing getting rid of the `@` sigil and possible smart ptr integration
- G: One issue that arose at the end of the discussion was figuring out how to denote a constructor expression that constructs its value into a given location
- G: Basically placement new in C++
- G: It seemed to me that a minimal change for permitting smart pointers would be adding an expression type to the language similar to placement new
- G: Takes two arguments (`alloc (provider) expr`, maybe?) where one of those expressions would evaluate to something that yields memory
- G: Am I on the right track here? Is this what people were thinking we would need?
- G: Other thoughts? It is an addition to the expression forms, but it possibly puts us in a position to punt out the sigils for pointer types, and improve smart pointer integration
- P: Replacement for the `@` expression so net quantity remains the same
- G: Generalization of `@` and `~`
- G: Going to want the ability to handle "shared allocators" 
- N: `new expr` <-- "~T", `new (prov) expr" `prov<T>`
- N: basically it would be nice if this could be *the* allocation expression
- P: niko and I were discussing something like `new Foo::init()` as the default way to create a heap-allocated object
- G: Winds up potentially entering the type system in that you have to have a trait that providers implement so you can parameterize over it
- G: You would want potentially to write code that works over both GC and managed box
- G: You'll notice that lots of C++ code has an allocator argument, I kind of expect that we'll grow it
- G: If you want to write an algorithm that's alloc independent, that's the use case
- P: I have given some thought to this, I don't know, because a lot of times it is hard to separate out the characteristics of the data structure from the custom allocator
- P: For example consider a binary tree, a binary tree where you can remove elements probably doesn't want an arena, since that would just leak
- P: But a binary tree where you can't remove would fit better, 
- P: doubly linked lists similarly are not suitable for all allocators
- N: this all sounds like something that post rust 1.0 would be well-suited to higher-kinded types
- G: Maybe has more detail than we'll get here, just wanted to summarize and poll
- P: I don't think we need a trait for this, all you need is that the provider evaluates to a function, but you just lookup the return type of that expression
- N: You need some way to get from the return type (which is uninitialized) to the location to fill in the result
- P: I've been meaning to talk a little bit today about simplifying the mut-borrowing story in regards to this, we may be able to effect a large simplification on the language
