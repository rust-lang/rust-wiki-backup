# Agenda 01/07/2014

* Do we support windows XP (acrichto)
* option api renames (acrichto) https://github.com/mozilla/rust/pull/10854
    * map_default => map_or
    * mutate_default => mutate_or_set
    * changes parameter ordering (default, fn) => (fn, default)
* guard pages (acrichto) https://github.com/mozilla/rust/pull/10460
* foo_opt => foo (acrichto) https://github.com/mozilla/rust/pull/11129
* loadable syntax extensions (acrichto) https://github.com/mozilla/rust/pull/11151
* static trait method syntax
* rc (brson) https://github.com/mozilla/rust/pull/10926
* friend of the tree (brson)
* pull request backlog (brson)
* abort on double-fail (brson) https://github.com/mozilla/rust/issues/910
* bounded channels (pcwalton)
* conduct: foul language on irc channels (pnkfelix)

# Status

- acrichto: vacation!
- brson: 0.9; docs; android
- pnkfelix: gc

## Friend of the Tree

- brson: Vadim Chugunov has contributed to Rust since June. He has done a lot of work that we have been putting off. He has fixed numerous bugs on Windows, including fixing our test suite. He also got us working with newer versions of MinGW. Recently, he also completely elmininated our dependency on C++ by rewriting our exception handling. Make sure to thank him!
- all: (clapping!)

## 0.9 & PRs

- brson: It's almost there. Need some more documentation cleanups. I want to move it into documentation entries from the wiki. The RC will roll either today or tomorrow, with the release following shortly. 
- brson: Our PR backlog is 44 long. They're just piling up and not making progress. If you can, filter them down. Also, we might need to add some more reviewers. Are there contributors we should consider?

## Should we support XP or not?

- pcwalton: It doesn't support condition variables, according to strcat. 
- acrichto: They have semaphores and mutexes. It's just really slow.
- pcwalton: We have to use a lock & signal mechanism. That's an argument for dropping XP. That code is more complicated than it needs to be in order to support XP. It's a direct translation of the C++ code and could be much faster without that complication.
- pnkfelix: Maybe just for VMs because XP was faster in them than newer versions of Windows.
- brson: Do we build on XP now?
- acrichto: Don't know. There were objections on IRC about not supporting XP, so I wanted to discuss it here. We could keep punting it for now. If it's just cvars, we don't need to completely drop it.
- brson: So we can change our lock & signal code if we drop XP?
- acrichto: Yes. Because they don't have cvars there, the code has to have more hooks and emulate it. In Vista and above, we have better options. That said, nobody should be using the condition code directly anyway.
- brson: There will be no XP buildbot, but there's no need to do anything specific.

## bounded channels

- pcwalton: We need bounded channels. The mailing list discussion was in great favor of bounded channels. The only real debate was just if unbounded channels should even be supported - nobody wanted them as the default.
- dherman: Experts agree with bounded as the right choice and that unbounded are dangerous and hard to fix.
- brson: Unbounded channels have the worst failure mode. When you have an unbounded send, you find out via OOM. Dealing with a bounded channel with backpressure, you don't want to deal with it. Is there some compromise here? Don't want call sites to deal with backpressure.
- pcwalton: Maybe go's solution? Block? There's some disagreement. 
- dherman: What's the concern about blocking?
- pcwalton: The failure condition is deadlock. SimonMar pointed out that nothing can proceed unless both sender and receiver are ready. So you can get some deadlock scenarios due to the implicit dependencies. In the async case, the sender can always proceed, but in the synchronous case, the sender can't proceed until the receiver is ready.
- dherman: This is hard. It's going to need some more work. It would be good to have somebody champion this stuff.
- jack: If we have both in the language, will we have to choose? 
- pcwalton: Maybe a parameter on chan, like None or Some(N) for the depth. Go uses a default bound of zero.
- acrichto: If you have to choose, it just becomes annoying (e.g., SharedChan). Different types because of different semantics is hard. If we have a blocking send, we'll need a select and all the related infrastructure. 
- pcwalton: Agreed.
- brson: We've talked about combining SharedChan and Chan. Wary of having a wide range of channel types.
- pcwalton: Danger with bounded channels that it's impossible to guess. Almost all are "0" or "10". It's like the argument to listen.
- brson: My preference right now is to combine Chan and SharedChan to add some "type" space. Then add an arbitrary (large) bound on the send. Then if people hit the bound, they can increase it. Then we just have a second channel type.
- pcwalton: Send it to the mailing list! Get the experts. 
- lars: one reason we'll see difference is that people that aren't on distributed machines like synchronous models and those in the distributed areas like asynchronous models.
- dherman: when you say distributed, are you including multi-process on a single machine?
- lars: yes. for the systems I've seen.
- dherman: for servo the multi-machine use cases are not the focus.
- pcwalton: any network service is a kind of distributed system. this is relevant to rust because there has been talk of moving unix sockets and tcp behind a single abstraction.
- brson: has anyone been arguing for synchronous channels?
- pcwalton: yes.
- lars: in servo we use synchronous things for various parts.
- brson: are we going to need three channel types
- lars: as acrichto said, you need more work to do sync channels.
- pcwalton: You can collapse them together with an argument on the constructor as well. Go's "bound" argument collapses the types together. It's like a mode argument.
- acrichto: A little wary of that because certain APIs might not be expecting to block on a send and some might. Even then, though, it would reduce the cognitive burden in terms of the number of types.
- brson: I'll write up my proposal and send it to the mailing list to see what people think.

## Conduct

- pnkfelix: Occasionally, the IRC channel has foul language. There's no policy against it right now, but can we consider officially saying that it's unprofessional / not appropriate? No bans, just discourage/warn about it. It's a slippery slope.
- pcwalton: Matter of degree. If somebody's frustrated, it might be fine, but if somebody is constantly using vulgarity to the point of annoyance, that's over the line.
- dherman: Want to avoid having an all-encompassing policy. Can we set norms? Have somebody ask to "keep it clean." Don't need to bring in policy, just try to encourage people to keep it open and welcoming. 
- pnkfelix: Yeah, nothing in the current policy right now.
- dherman: I mean, can we avoid having an explicit policy for now and just use gentle persuasion and see how it goes?
- pnkeflix: I need backup on this so people support me or at least don't think I'm over the line.
- dherman: I personally don't care about that. I care more that people who are talking down about other people's work leads to toxic negativity. So, foul language isn't as big of a deal to me, but it is to some. Different constituencies feel differently.
- pnkfelix: Agreed. 
- azita: Have you tried to intervene?
- pnkfelix: No. We don't have a policy. And if the norm is that it's OK, I didn't want to say anything.
- dherman: Just looking at it, I'd say that a, "please keep it clean" is totally reasonable.
- brson: I think it's appropriate to ask people to clean up their language.
- azita: But let's not force pnkfelix to just handle it all.
- brson: I'll try to keep an eye out.

## double-failure

- brson: I made a decision on failure during unwinding. Now, if that happens, we "abort with authority." It explains why we're doing that. We can change it eventually, but at least for the short-term, abort on double-fail is good.
- pcwalton: Works for me.
- jack: That's how it's worked for us so far. I haven't seen any problems in Servo with that.
- brson: Same as C++.

## Renaming options

- acrichto: "Default" should not be a word in the Options API PR. The further contentious thing is reordering the arguments from (dflt, fn) to (fn, dflt). I like the drop of the word default, but not the change.
- brson: The convention is to keep it the function at the end.
- jack: Erlang is for function at the beginning.
- pcwalton: Can we look at other languages for precedent?
- dherman: You'll drive yourself crazy looking at other languages for precedent. 
- acrichto: And the syntax would look like `map_or(||a,b)`, which looks ridiculous. I'll say the name change is fine, but the order should stay.

## Rc ptrs

- brson: PR for a change to Rc and a new version... I don't like what we're doing.
- pcwalton: I talked about it yesterday. At first he wanted weak pointers added and I said to make it separate, but that's a mistake. Having weak built into Rc pointers probably doesn't matter that much. If you're optimizing that much, you're using `unsafe` instead of Rc anyway. So, there should just be one Rc type, which is what he calls Strong. Rc should be a Strong, ref-counted pointer. Should support weak pointers. You can create weak pointers from a Rc type, then. There's some more overhead, but only affects space a little bit and time only at shutdown. I'm not convinced the overhead is that bad, especially for people who have elected to use Rc.
- pnkfelix: What is the type you get back?
- pcwalton: Weak<T>
- pnkfelix: So there are two?
- pcwalton: There were Strong, Strong-supportingWeaks, and Weaks. Getting rid of the first one.
- acrichto: If we're comfortable with just one Rc type, that will help us moving forward.
- pcwalton: Makes doubly-linked lists easier. There are some issues with Rc, especially around mutability, right now. There's a separate question of what bounds you want, but let's not tackle that now.
- brson: I think we want to remove restrictions that prevent cycles on Rc, right?
- pcwalton: Yes, especially when you have weak pointers.
- brson: So we don't statically prevent cycles. You'll have to use weak pointers strategically.
- dherman: If you have a cycle, you have a loop. The failure model here is a memory leak, which is not preventable statically entirely, so this is just a thing that programmers have to not do.
- pcwalton: There are a lot of ways you can leak in Rust e.g. spawn tasks that livelock.
- brson: All comfortable with this expense?
- pnkfelix: Yup.
- brson: I like this.

## guard pages

- acrichto: Spawning threads with guard pages is 3x slower than current. It's much safer (c code can't overflow if you call into it). Do we want to take this performance hit? In general, task spawning isn't optimized right now, but this is an even larger step backwards. Other opinions?
- pcwalton: We're never going to be that fast at task spawning. Our typical strategy has been to allow users to turn off the safety if they need the performance. Could have a `dont_map_guardpage`.
- pnkfelix: Or an unsafe entry.
- brson: Going to need a task caching strategy. Would that save us anyway?
- acrichto: That's also added.
- brson: But it didn't help?
- acrichto: Not sure. The benchmark was just for spawning lots of tasks. It's also a per-scheduler cache.
- brson: I'm kind of in pcwalton's camp. We need the safety. And we need to make task spawning fast. Might as well land it and optimize later. Does it work on windows?
- acrichto: Yes.
- brson: So, we now have guard pages and `more_stack` prologue, right?
- acrichto: Yes.
- brson: Do we really care about C overflowing the stack?
- pcwalton: MSVC. Added during the big security push.
- acrichto: Linux also has the option.
- pnkfelix: Can we get rid of the more_stack prologue? 
- acrichto: Would require changes in LLVM. It's certainly not easy.
- brson: Would be a shame to have two strategies here. Even if we do touch pages, we're still going to need to have branches in teh function prologue.
- pcwalton: We don't need the branches. LLVM does it for MSVC on that platform and hopefully it will come to other platforms. Maybe we can take it then post-1.0.
- acrichto: Take the safe default for now. I'll add an issue... wait, this doesn't work on native.
- brson: Can't mprotect part of our stack?
- acrichto: We don't know where it ends. Well, we ask pthreads for an amount and know where it starts, but we don't exactly know what the last page is for sure.
- brson: Let's think more offline about this.

## loadable syntax extensions

- acrichto: PR with a lot of the infrastructure. The idea is `extern mode` and say `phase syntax` on an attribute and the compiler will try to open it up to find it. The compiler calls dlopen and blindly pulls the metadata out of the other crate for you to use. The problem is that the version of libsyntax isn't verified, but if everything is linked against different versions this could blow up. Behind a feature gate right now. It's also not clear how we're going to move standard macros into a library. If there's an idea of it being exported, the macro has to be in a separate crate. So we'd have to pull it out. Others' thoughts? I'm tempted to merge it because it's behind a feature gate and it's great work. It's not close to ideal yet.
- dherman: Few thoughts (and an active baby...). Macros, modules, and phases are really tricky. There is a high probability that the patch is wrong because this is just an amazingly hard problem. It's fine with landing and feature gating, but there's a high risk that it will have to be significantly changed. I was hoping to sit down with pcwalton to talk through what the patch does and what our plans are. I'd love to get involved in this because it's my core area of expertise.
- pcwalton: If there's something wrong, it's my fault, since I directed a lot of it. I'd like to talk more about what phase `extern mod` is supposed to go into. I think explicit phasing is important to avoid the C++ slope of compile-time function evaluation with ad-hoc stuff. Phasing will save us pain down the road and complexity, but that said, I'm not sure the design is right. I still think we should just merge it because it's in line with what dherman and I have been talking about before, so it should be on the right path.
- dherman: When I said it's likely to need to be changed, it was an uninformed guess. Nothing concrete that I know is wrong with it, and we share our inclination. 
- acrichto: I'll defer the decision to pcwalton. But we should merge this somewhat soon if we can.
- pnkfelix: I'm in favor of merging, assuming it passes review.

## Methods that fail

- acrichto: There are a lot of methods that fail and have a foo and foo_opt version. There's a PR to remove all _opt versions and just have a foo version that returns the option. So, it's pushed onto the user. It seems like a great idea to push it to the user, but it feels like the language is still at fault because .unwrap is already pretty prevalent. If the norm is unwrap, then it hasn't helped us. The question is: do we want to avoid doubling APIs with _opt and failure? And should we have a better way to deal with options coming from APIs? Or just deal with what we have?
- pcwalton: I feel like every other functional language just duplicates the APIs because it's so convenient. Second thought is, can we change `unwrap` to `get`?
- brson: I'd like consistent guidelines for unwrap, get, borrow, and get_ref. We use get for other things (where it gets a reference).
- dherman: The whole overall API design principles thing is still lacking. 
- brson: We should solve it by 1.0.
- pcwalton: I like get because it's shorter by three characters. Maybe all the gets that return references should be borrow. I feel like I don't really care much about the method duplication. It's so convenient.
- pnkfelix: I'd like the opt version be the shorter version. So, foo and foo_failing.
- brson: There was a big thread on this at some point, but I don't know if there was consensus...
- azita: We're out of time!
- dherman: An operator for get like prefix-?
