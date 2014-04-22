# Agenda 4/22/2014

* regular expressions (acrichto) https://github.com/rust-lang/rfcs/pull/42
* attributes on match arms (acrichto) https://github.com/rust-lang/rfcs/pull/49
* rev_iter() (acrichto) https://github.com/mozilla/rust/pull/13648
* f128 (acrichto) https://github.com/mozilla/rust/pull/13415
* integer fallback (nmatsakis) https://gist.github.com/nikomatsakis/11179747
* Favoring Vec<T> or ~[T] (pnkfelix) https://botbot.me/mozilla/rust-internals/msg/12851440/ 
    see also:   http://www.reddit.com/r/rust/comments/21y9tw/meetingweekly20140401_vect_vs_t_virtual_structs/cghztkv  
* unsized/type/Sized? keyword (nrc)
* bounds on type vars in structs and enums (nrc) https://github.com/rust-lang/rfcs/pull/20
* module imports (brson) https://github.com/rust-lang/rfcs/pull/18
* rfc drawbacks (brson) https://github.com/rust-lang/rfcs/pull/32
* remove markers (brson) https://github.com/rust-lang/rfcs/pull/12
* smaller refcounts (brson) https://github.com/rust-lang/rfcs/pull/23
* friend modules (brson) https://github.com/rust-lang/rfcs/pull/47
* disablable asserts (pcwalton) https://github.com/rust-lang/rfcs/pull/50

# Attending

- brson, azita, aturon, acrichto, jack, pnkfelix, erickt, nrc, larsberg, niko

# Status

- acrichto: bots bots bots, llvm 3.5, upgrading libuv, timeouts 
- pcwalton: Servo bugs, opt-in built-in traits
- pnkfelix: Control Flow Graph; Lifetime bugs
- aturon: float constant bugs, a bunch of other low-hanging fruit (warmup tasks)
- nrc - DST
- brson: proj management, intern prep, docs, contracts, 'box', VPC
- nmatsakis: refactoring in support of #12624, zeroing, and some other stuff

# Action Items

- acrichto: merge https://github.com/rust-lang/rfcs/pull/42
- brson: merge https://github.com/rust-lang/rfcs/pull/32
- brson: close https://github.com/rust-lang/rfcs/pull/18
- nrc: merge two RFCS struct typarams, get niko to merge
- acrichto: merge https://github.com/rust-lang/rfcs/pull/49
- brson: close https://github.com/rust-lang/rfcs/pull/47

# Welcoming aturon

- brson: aturon has joined us in SF!

# Friend of the tree

- brson: klutzy has been doing an amazing amount of Windows work for years now. He picks up issues that affect our quality on Windows and picks them off 1 by 1. It's tedious and doesn't get a ton of thanks, but is hugely appreciated by us. As part of the Korean community, he has also done a lot of work for the local community there. He is a friend of the tree. Thank you!

# regular expressions RFC 42

- acrichto: Adding regular expressions to the standard distribution. It's all written in Rust. Based on re2. Looks really well-written and is impressive all-around. It's something we'd like for 1.0, but won't block on. This looks pretty good to me and I'd be in favor of merging this.
- brson: Also passes a major re test suite.
- acrichto: and it's not slow!
- nmatsakis: Does it compile them?
- brson: Just bytecode, not native code.
- pnkfelix: The compiler that it runs at compile-time for the regex macro is indistinguishable from the dynamic call, and the public API does not distinguish between compiled at runtime vs. compile-time. Does that seem right? Shouldn't one with an invalid input have a dynamic error?
- acrichto: Only at creation.
- pnkfelix: Does the public API here only offer the regex! macro?
- brson: There's a crate that does everything at runtime. A separate one compiles the regexes to bytecode. There's also a third crate (not for merge now) that generated native code at compiletime.
- acrichto: There's also a dynamic API that offers a string.
- brson: He believes we can offer this backwards-compatibly.
- nmatsakis: Anything unicode-y stuff?
- acrichto: It's in the rustdoc info on the PR. Not doing normalization. I think it should be considered experimental, still. 
- nmatsakis: Agreed, bring it into the 1.0 standard library, but not as stable.
- brson: Basically everything that goes in from now on will be experimental. Can we try to make this a submodule? It seems like a good test for the process of bringing in community modules...
- nmatsakis: We've been trying to break apart crates... what does it mean to be in the standard distribution?
- acrichto: In the Rust repo.
- nrc: Yes, and if we change something, we have to fix it, rather than them.
- brson: We do have to give some guarantees... if it's officially supported but not part of the main distribution, we'll run into the problem Servo has, where you have to update all of the submodules before you can update Servo.
- jack: The communicty keeps submodules in sync with Rust master, but Servo is not.
- brson: Right, but the change would be to change the rust compiler change, push the crate change separately, etc...
- jack: If bors had support for this process, it might be less cumbersome.
- erickt: Do we intend to use the regex as part of librustc or libsyntax? If we don't, then we could just only rebuild it after stage2, which might make life easier.
- acrichto: Every library not used by librustc is only built as part of stage2 as well.
- brson: Maybe we don't need to get bogged down in the details; it sounds like everybody is good with this RFC. Who will merge it?
- acrichto: I will.

# integer fallback

- nmatsakis: Should we retain the implicit integer literal having a type variable bound to an integral type and, if there are no constraints, we pick int. It's nice that you can write simpler code, but it can result in unexpected types / bugs. I went through and made soem changes for what would happen if we remove this behavior (causing an error) and summarized the results in the gist. Also made lshift and rshift take uint. And made the repeat counts on an array uints. The type inferencer also got a change where if you have an integer literal where if you take it and cast it as another type, today it's considered unconstrained, but what I did was propagate down casting those to a u8 to choose a constrained type. `0 as *T` was common. Also, across the int and uint modules. Once I got rid of those, I added a warning, and as you can see there aren't a lot. Enum discriminants and ranges and counters. `for _ in range(0,20)` has no constraint on what kinds of integers 0 and 20 are. Also counters where something is initalized at 0 but don't do much with it. The last case is macros that replicate the code a number of times. I was encouraged by this, since it did not seem like massive numbers of compilation errors in rustc. There were only around 20 places to fix. In enum discriminants, if you write `Red = 0`, we don't constrain it. We might want to define them as integers. It's weird; C is very vague. It says that you should pick something that is at least big enough to store the integers that are in there.
- pcwalton: Isn't there something where you can say what size it is?
- nmatsakis: You can put a repr annotation there.
- brson: That's scary to me. Attributes should be metadata....
- pcwalton: repr is already not metadata.
- acrichto: No code will compile differently based on repr, though.
- pcwalton: Transmutes called on different sizes are an error, though!
- nmatsakis: A repr will change the size of your types, which can cause a transmute to fail. I guess we could pick and have you pick a specific integer type...
- brson: could trivially say it's an int and truncate?
- nmatsakis: The problem is that there's no perfect choice. If they pick u64 but use a bigger constant, it'll just get truncated. I don't really like this feature to begin with. But these would not be cases where manual annotation is required, no matter what we choose.
- acrichto: Is this work worth it despite the drawbacks?
- nmatsakis: It's just ranges; I felt like it was really minor. The closest things I found to bugs were using ints to count things in an array, which can never be negative.
- brson: Right, it could overflow and cause havoc?
- nmatsakis: Hrm, that would be a big array... probably never going to happen. I'm in favor of continuing to investigate this.
- brson: pcwalton?
- pcwalton: I'm fine with it. I was opposed previously because I assume there would be a lot of errors. Range is concerning, but not enough to say no.
- brson: Yeah, it's a wart.
- nmatsakis: I rarely use n fixed iterations...
- acrichto: Yeah, it's stuff people run into all the time when they first come to Rust.
- nmatsakis: It's only going to happen when there is a fixed number in your range; it will infer correctly if your numbers are based on the length of an array.
- brson: Write up an RFC?
- nmatsakis: OK.

# rfc drawbacks RFC 32

- brson: Let's add another section to the RFC called drawbacks: yes/no? Awesome, I'll merge it.... objections?

#  Favoring Vec<T> or ~[T] 

- pnkfelix: So, ~[] is not going away, and there was a whole thread that happened. The core queestion is: what is our philosophy? Should we argue in favor of using ~[] in many places or not? The piece I understand/respect is that when you're returning a vector you built up, you can't predict if the caller will want to push more elements or not, which is an argument in favor of Vec. On the other hand, ~[] syntax carries more information, in terms of it's intended to be a fixed vector. How do we make this judgement call?
- nmatsakis: I don't feel ready to make this call 100% until more pieces are in place. 
- pnkfelix: One possible answer is to say that we're still researching this.
- nmatsakis: I still feel ambivalent. I am open to there being options for returning Vec. I personally like to return ~[], but can recognize that might be the right choice for some functions.
- nrc: And ~[] is not going away, since it falls out of DST. 
- pnkfelix: Some want that to be the reason for why ~[] exists; it's a consequence of DST, but not something you should use.
- nmataskis: Since you can convert between them so easily, it's hard to care much.
- pnkfelix: But that issue gets into whether we trim or not upon conversion to ~[]. 
- nmatsakis: So, I think there's a call to trim, but it depends on the allocator to decide whether that does anything. I suspect the default allocator will no-op trim.
- brson: Nobody wants to make new decisions on this subject...
- nrc: I think we should wait and C. We'll have to make a philosophical call for our own libraries, but as long as we're not abandoning one of them right now, it's fine.
- nmatsakis: I'd prefer that people not go rampantly changing ~[] to Vec... though that did honestly already happen to some extent.
- acrichto: ~[] is no longer growable.
- pnkfelix: I just wanted to make sure we talked about it.

# disablable asserts

- pcwalton: We need to be able to disable asserts. My original suggestion in the RFC to have assert and enforce was not received very well. There are several options here. Can have assert be a function. Or assert! be a macro.
- nrc: What was the suggestion?
- pcwalton: Some don't like to write `enforce` in tests and think assert should be written in tests. And then if assert is disableable, you can't test it.
- brson: My concern is that assert *is* the word for this. Assert is the default word they will use, whether it's in tests or in logic. An assert that is always on is guaranteed to always give you correct behavior.
- pcwalton: I disagree about it in logic!
- brson: If assert never turns itself off, it will always be safe to leave it enabled. It's doubly safe because it's historically Rust's behavior. I'd prefer to add another macro to disable it.
- pcwalton: It violates least surprise, as assert is always disableable in program logic.
- nmatsakis: People just don't run tests with asserts turned off.
- pcwalton: There's a syntactic difference in java, certainly. you use a method call in tests and a statement in program logic.
- nmatsakis: So, assert is usually something that can be disabled. I think we need a name. I don't care if it's assert/debug_assert/enforce, etc. I sympathize that if you come from the outside, you'd expect assert to be able to be turned on/off.
- pcwalton: We need to support some kind of assertions that can be disabled in the standard library. I see no reasonable objections to that.
- nmatsakis: brson, do you have a preferred name for the disableable assert?
- acrichto: Can we leverage macros to use the same word?
- nmatsakis: I don't know...
- pnkfelix: What does that mean?
- nmatsakis: You mean what you write inside the macro tells you if it's always on? 
- nmatsakis: `assert!([debug] ...)`? Something like that could work.
- erickt: Just a ! as the difference seems scary.
- nmatsakis: I think if the assert keyword had been around forever, junit would not have used those macro names
- brson: One thing I like a lot is that the line between test/code is blurred. You just build it into the compiler. I like using the same assert in both places. I would not be opposed to having a different set of asserts to use in test code. It's hard to message at this stage, though.
- nrc: We'll need something different... 
- nmatsakis: If we had assert and debug_assert, I guess everybody would be happy. (by everybody, I mean on this call)
- acrichto: Some will get unhappy if assert is the one that is disabled by default. Some will get unhappy the other way.
- pcwalton: It's a much more minor issue than not having any disableable asserts at all in Rust.
- brson: In the spirit of making progress... let's start by adding debug_assert. It's almost trivial/free to do.
- pcwalton: +1
- nmastakis: We'll play on our "safety-oriented language" title for why asserts are on by default!
- brson: pcwalton, can you update the RFC?

# module imports RFC 18

- brson: I'd like to close this.
- nmatsakis: Agreed.

# bounds for type parameters on structs

- nrc: Any objections?
- acrichto: You did bring up that adding vs. enforcing were distinct...
- nmatsakis: We're not going to add them without enforcing them, rigth?
- nrc: How we want to enforce the bounds on type variables... having them on structs doesn't help anything because of the way that we check them. It's a bit of a chicken and egg. Either we enforce them first and then a bunch of stuff won't work because you can't annotate them there. Or you add them but don't get them checked first and that's also a mess. 
- acrichto: In terms of implementation, it might need a few phases. But I'd assume the RFC is for both.
- nrc: The enforcing bit isn't just about structs. It's also about enforcing everything else. I guess the RFC for struct bounds is... I guess we wouldn't take this without the other parts.
- nmatsakis: I don't really follow. Are you saying, acrichto, that we should take the two together?
- acrichto: Yes. It seems non-helpful to be able to write bounds but have nobody look at them.
- nmatsakis: I agree.
- nrc: I guess if we weren't going to take the other RFC (to check the bounds properly everywhere).... I guess we have one RFC to check the bounds on structs. We could accept this. But then the other RFC that checks the bounds everywhere is subsumed by that. We could have one or the other without both. But it makes sense to do them as a package.
- pnkfelix: Are they ready to be accepted?
- nrc: I believe so.
- nmatsakis: We should take them both, and if we want to check at a more limited set of locations later, that's a minor RFC. So long as the invariant continues to exist where you cannot have a struct whose bounds are not met, I'm fine.
- nrc: The other RFC is #34.
- brson: Accept both of these today?
- nmatsakis: I am.
- acrichto: Can you close 34, put it in the other one, and we can merge that one?
- nrc: Yes. Though I might do it the other way, since 34 is the more fundamental one.
- brson: You're going to merge the two RFCs and then somebody will land them. Niko?
- nmatsakis: Sure.

# attributes on match arms 

- acrichto: We talked about this last week. Is it OK to merge?
- nmatsakis: Go for it.
- brson: Just on match arms? Great. 
- acrichto: I'll land it.

# friend modules - RFC 47

- brson: I do not believe we need this feature for 1.0.
- nmatsakis: You can carefully use `pub use` today to handle sharing between a set of modules.
- nrc: Hopefully we could add this backwards compatible post-1.0, if we decide we need it.
- brson: I will handle summarizing feedback and closing it.

# f128

- acrichto: It would be behind a feature gate. There's no RFC, but it's pretty uninvasive. 
- pcwalton: Maybe make it experimental. f128 is here; machines with it have shipped (I don't know where they are, but they exist). z-architecture for IBM mainframes have it. Some other SPARCs that haven't been manufactured yet have it.
- nmatsakis: What happens if you use it and there is no processor support? Does LLVM break it down and use it?
- pcwalton: __float128 has existed in gcc for a while, so it works. It's not native to the processor, but it does work.
- nmatsakis: Emulation routines?
- pcwalton: It must. I'm leaning towards approving this because it's already landed in LLVM and doesn't sound like a huge maintenance burden. 
- acrichto: We'll keep it behind a feature gate; don't even need to make it experimental.
- pcwalton: It's in the fortran spec, so any compiler supporting fortran must support it. So I'm guessing all backends will support it. My only concern is that I don't want it to be a stupid feature that nobody uses that prevents us from porting Rust to hardware where there's no emulation library.
- acrichto: This does enable it in the standard library because of the visitors, so we need to make sure it doesn't bring in a dependency, but otherwise I'm ok.
- brson: OK, let's do it.
