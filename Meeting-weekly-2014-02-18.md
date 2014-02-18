# Agenda 02/18/2014

* install media (brson)
* debug assertions (acrichto) https://github.com/mozilla/rust/pull/12108
* weak extern fns (acrichto) https://github.com/mozilla/rust/pull/11978
* if_ok! (brson) https://github.com/mozilla/rust/issues/12037
* channel names (brson) https://github.com/mozilla/rust/issues/11765
* error code names (brson) https://github.com/mozilla/rust/pull/12144
* Hash (brson) https://github.com/mozilla/rust/pull/11863
* wontfix fixed-length vector syntax (brson) https://github.com/mozilla/rust/issues/9879
* unsafe ptrs (brson) https://github.com/mozilla/rust/issues/7362
* &, &mut and multithreading (brson) https://github.com/mozilla/rust/issues/11583
* issue numbers in commit messages (nrc)
* relative ref // req leading `::` in use (pnkfelix) https://github.com/mozilla/rust/issues/10910

# Attending

- brson, pcwalton, acrichto, larsberg, nmatsakis, nrc, metajack, dherman, pnkfelix, cmr

# Status

- acrichto: rollups, landing things, rustdoc fixes, now working on rewriting std::run
- brson: installers, atomics, unstable, docs
- nmatsakis: nothing this week -- unless I find time to prepare some RFC details
- nrc: burn down bugs, associated items proposal
- pcwalton: wrapping up servo stuff, removing ~[T] from the language

# if_ok!

- acrichto: Committed if_ok and then opened issue to rename it. Leading is `try`, but people are worried about `try` / `catch`. Have check, result, OK, things like that. My favorite is try.
- brson: Also has the most support. Everybody okay with `try!`?
- pcwalton: Sounds good.

# issue numbers in commits

- nrc: I proposed we stick issue numbers on the commit message because it's hard tracking down the comments on a commit that broke code when I tried to rebase. In FF we always put a bug# on the commit message, but the mailing list seemed like people were not in favor of it. Huon posted a script to make it easier to get back to the actual issue instead of the PR. Maybe just use that? Or should we require issue numbers? One thing I didn't realize is that most PRs do not have an associated issue logged with them. That makes the proposal less appealing.
- pcwalton: Because PRs and Issues are unified (every PR is an Issue), it's hard with GitHub. But the weird thing is that you can't transform an Issue into a PR easily (without the API). If you could transform it easily, this would be no real problem...
- brson: If there's an issue, it should definitely be tagged in the PR. It also seems like we would like people to start having issues backing up their PR more often. Today, we often get PRs that have not had discussion before code was proposed.
- nrc: I'd thought that we should have an issue for basically everything because for code to be so trivial that there is no discussion required about it seems rare.
- nmatsakis: Comments, typo fixes, etc. might not need an issue. I like how it's managed in Gecko. Very orderly, and I wouldn't mind having some structure like that.
- brson: We could start by simply asking people to justify larger PRs with an associated issue...
- nmatsakis: Start with "if you have an associated issue, please put it in the title of the PR." then it will be the title of the bors commit.
- pnkfelix: Not quite right; there's a bors bug that leaves out the title, but we should get that fixed.
- nmatsakis: Hrm. Maybe not so helpful, then...
- nrc: Could we have bors search the title and first comment side for "Fixes #N" and automatically put that in bors' commit message? I don't know much about git/bors.
- acrichto: Bors' message is based basically on the single-commit description or the manual description if multiple commits.
- brson: Could have bors give you some feedback on your pr, like they do in Servo.
- jack: When bors merges a PR, it could maybe add some notes for each of the individual commits. Would that help? Notes are metadata that can be attached to a commit. Not sure if they show up in a way that's useful, but bors could certainly add notes to all commits that it processes.
- nrc: That sounds good, as it keeps issue# in easy access. As long as it doesn't require git arcanery to track it down.
- pnkfelix: Another aspect is that if you use git blame, it doesn't tag a line with the bors merge but rather the original commit message. You'd still need it there instead of in the bors merge in order to be able to find it.
- jack: Should see if the github API shows the notes there.
- nmatsakis: git show should show the notes.
- jack: And git log should show it, too.
- acrichto: Perfect. bors could just say 'this commit is associated with a certain PR'
- jack: Two things. Annotating the commit. Also the quesiton of discussing the work (having an issue before a big PR).
- nrc: Yeah, that's a separate question. But if there is an issue there, can bors find it from the PR? Or do we need some convention like "closes X"
- acrichto: Sometimes multiple issues.
- nrc: That's fine; a list of issues is OK. You'd have to have that anyway to know what the discussion was.
- acrichto: For every commit, scrape all the issues mentioned in the PR and put them in the notes for each commit.
- brson: Any conclusions here?
- nrc: Get bors to add notes to every commit? Sound like a good plan?
- pnkfelix: I don't know much about notes. Not clear if with merged commit or the original ones or both.
- nmatsakis: Need it on both.
- brson: I worry because I've never heard of notes before now, and we're about to rely on them a lot.
- pnkfelix: Maybe explore first before changing bors to make sure it's not expensive?
- nrc: Yes. Need to sort of have it in place before we can really make a call on its usefulness. Is there someone who knows how bors works who knows how much work it is?
- jack: If adding notes is in the github API, it's easy. Bors only communicates with git through github; it doesn't perform git actions natively...
- brson: bors does push, at least. So it could use some git locally.
- nmatsakis: Seems like notes are new, and don't get fetched by default. Seems like just asking people to use issue annotations is more straightforward, though laborious for lots of commits.
- nrc: Personally haven't found it laborious in Gecko, but maybe other people do.
- brson: I'd rather people put those issue numbers in the commits themselves. Seems like where they belong.
- nmatsakis: Say that commits should start with "Issue # -" or "No issue -"?
- acrichto: I've seen this with commit templates before. And bors can reject the commit if it doesn't follow the template. Would also make the commits have a uniform appearance.
- pnkfelix: Just a rule about the first line/sentence? I'm fine with saying commits have to start with Issue# / No issue. May want to also have the notes step independently.
- nrc: And you're fine with not requiring an issue#?
- pcwalton: Worried about the effect on community contributions because it's not standard GitHub practice.
- nmatsakis: Given that we can edit PRs, we could just tag it ourselves...
- brson: We need a decision. What are we doing? Request that people add issue numbers to all their commits?
- pnkfelix: Spend a week looking at notes option first?
- brson: One issue is that I don't want to do a lot of automation work here. I don't want to spend a week doing anything.
- pnkfelix: If we decree something and the community says, "how about this other way?"
- nrc: Maybe suggest a template? See if it's a problem for people? If not, then make it mandatory.
- nmatsakis: How easy is it to make a template?
- brson: Minimum we can do is have bors warn them if they don't post a template.
- nrc: That seems like a good incentive.
- brson: Sounds good;we'll get bors to warn people about not putting the issue number in commit messages (not just the PR).

# debug assertions

- acrichto: Huon opened a PR to add debug assertions. At the same time renamed assert to enforce. The idea was that assert in C can be turned off with -NDEBUG. With that, the idea is that assertions should be assumed not to be on by default. Everybody agrees that there should be both "debug only" and "appears in all configurations." But there's no consensus on names. Enforce always there; assert compiles away. Or assert always there and debug_assert compiles away.
- nmatsakis: Longstanding practice of assert as the one that gets compiled away. 
- pcwalton: Yes, aborting in production is something we should support, but it should have a different name. Graydon wanted it to be impossible to turn off asserts. In practice, what happens is that I don't use assertions (in Servo) because I know they'll be slow. 
- nmatsakis: D uses enforce for always check this.
- acrichto: I'm a huge fan of assert and want the always-on to be called assert. I know that in C, you can turn it off with NDEBUG, but I always program with them. If we're clear that it can't be turned off and there's an alternative that can be, that would be great.
- dherman: In FF, they use asserts liberally because they know it's a zero-cost thing. People would be just afraid enough to add asserts that they would do it far less often if they knew that it was not zero-cost. Not arguing against having the main way be always-on; just that we definitely have real experience in production code where people rely on zero-cost way of doing asserts. I don't officially have an opinion here, but want to make sure people know there's millions of lines of production code with this.
- pnkfelix: No assert for 1.0 and fail_unless and debug_assert?
- acrichto: No, we need something better than fail_unless
- pcwalton: I like that name, but everybody seemed to hate that name. But I caution about gecko as a data point, as they have non-fatal assertions.
- nrc: Now they have fatal and non-fatal assertions. Sometimes the non-fatal assertions are useful...
- pcwalton: But NS_ASSERTION is non-fatal, which is weird.
- nrc: Yes, weird. MOZ_ASSERT is fatal. The name is totally non-intuitive!
- nmatsakis: Any language where it's always on? Definitely not always-on in Java. Maybe python?
- cmr: There' s a flag to disable them. ("-O")
- nmatsakis: I don't know if any language where assert is always on, so my opinion is they should be off in release by default.
- brson: This will be hard to change.
- pcwalton: Just do a global renaming all assert to fatal_assert b/c that's how it works right now. One-liner perl script.
- brson: Change it for a while until everybody realizes that the change has happened...
- nmatsakis: Maybe need a lint? We need a policy for making these kinds of changes.
- acrichto: If we go with assert being the one that can be compiled away, what's the variant?
- pcwalton: I like fail_unless...
- pnkfelix: enforce has a tiny bit of precedence
- acrichto: Presented with enforce vs. assert, how do you know what's compiled away?
- pcwalton: Have to know that assert is compiled away in every other language.
- acrichto: Might not have that precedent?
- pcwalton: Precedent here is unanimous. Assertions can always be disabled. All ruby libraries that provide assert even allow them to be disabled.
- pnkfelix: So the failure mode is when you've used assert instead of enforce? So maybe make it hard to turn off asserts?
- pcwalton: Yeah, makes sense that turning off asserts should be a separate flag from optimization.
- acrichto: Maybe require?
- pcwalton: Sounds like you're importing a module.
- nmatsakis: People will just have to read the docs. 
- pcwalton: Assertions that can't be turn off are non-standard, but we have to come up with a name for it.
- brson: If we have a name, people ar emore likely to not use it. People use assert under the assumption that it can be turned off doesn't break code.
- pcwalton: Slow can break code.
- nmatsakis: js engine is littered with assertions and runs quite slowly with them on.
- pcwalton: I think assert needs to be able to be turned off.
- dherman: Not sure most compelling argument. People want the syntax that feels the most familiar. If we tell people the main workflow is asserts that can't be turned off, then it makes sense. But that's not the main workflow. Main is that you can't turn it off. My personal experience in spidermonkey is that asserts are like checked documentaiton. There was a case where i was building on invariants that weren't well-understood and tehre was a way to fail gracefully, and there I needed a dynamic failure. I think the more compelling argument here is that we don't expect to have a feature that is not disablable and used liberally.
- nmatsakis: Why not just write code instead of having `ensure`? Just if with a `fail!`?
- dherman: People do defined ad-hoc macros... but we can avoid having confusion.
- pcwalton: Three lines in rust because you need to brace if-bodies.
- nmastsakis: If we make assert disableable with a separate flag from -O, then it's the user's choice. Definitely wrong to have side effects in an assert, though.
- nrc: So, we agree that assert should be turn-off-able. Just need a name for the other thing?
- pcwalton: Not sure everyone agrees. Objections from brson/acrichto.
- acrichto: Not a fan of being able to turn off asserts, but that's the minority opinion.
- brson: Just worried about the philosophical about-face. Definitely need to do it now and not later if we are. BIG change to the feature. Have to audit every one of our asserts.
- pcwalton: Don't think we have to audit; think we can mass-change assert to enforce...
- brson: IF we keep the macro for the second thing. If we think it's not important enough...
- pcwalton: I really disagree with that philosophy (asserts always on) in Rust because it leads to people not writing asserts. 
- nmatsakis: See it in this HashMap PR. There's a big resistance in performance critical code to putting in assertions. I think I'd always assumed they were disableable, even though I knew they weren't...
- pcwalton: If you're not doing benchmarking, you don't notice. In Servo, I've had to remove lots of asserts.
- nrc: I didn't realize they were always on.
- brson: So, rough consensus. Rename assert to something else temporarily? Or just flip the switch?
- acrichto: I think just add the if cfg(not(debug)) to the assert macro and be done.
- brson: First would be to add enforce and mass-rename our asserts.
- acrichto: Do we want enforce in the standard distribution?
- brson: I'd be OK with not exporting it.
- acrichto: If you have two similar things, like dherman said, everyone will be confused.
- nrc: The more I like fail_unless instead of nothing at all.
- acrichto: I'm OK as long as it's not exported.
- brson: So, add private fail_unless macro and convert, then flip the switch on assert and tell everybody to be careful? 
- pcwalton: Don't have to be that careful. Just have to know you can turn it off. 
- nrc: People have to make the conscious decision. The behavior is only going to change if they add the config flag to their commandline.
- pcwalton: I feel like I have to be more careful with the current setup to make sure it's in a hot spot.
- brson: The issue is with breaking existing code.
- acrichto: Currently, we're breaking code all the time.
- pcwalton: Not that concerned about breaking existing code right now.
- brson: We have a plan. Private macro for internal use and then a special config flag that disables asserts.

# Redesign of the hash trait

- brson: No specific comments, but this is a very important trait, so if you hvae comments, get them in.
- acrichto: Right now, you have a trait hash and a trait hasher and a trait hash factory. So a hash map is created with a hasher factory. Every time you want to hash a thing, you create the hasher and call it. you can just impl Hash, though. We found out there's a custom trait for feeding things in, but we found out there's just an interface for feeding bytes in. The question is: what do you think about merging an IOWriter with this hashing API? So the signature is hashing yourself into the writer with conveient methods for floats, slicing, etc. The downside is that it all returns an IOError that you are told to use, but it removes duplication with these hash-specific types. If we did that, we'd reduce the trait explosion (just hash and hasher).
- pcwalton: I like that idea, and that's a great way to bring it down. It's also easily understandable. A hash trait where you've got a writer and have to put things into the writer makes total sense. Question: Does the writer have vtable dispatch?
- acrichto: No.
- pcwalton: How do I make a hash table on intern strings that uses the pointer value?
- acrichto: Custom writer where the intern strings just give away their pointer values ( a sum hash). So it'd be a hash of the pointer value itself.
- pcwalton: Same for integers?
- acrichto: Yes. I think you wouldn't do a literal ID to just return it. But a sum should work, and LLVM would do it.
- pcwalton: Use custom writers? If so, then you could have the "last" writer. THose are the big cases for Servo; interned strings for selector matching. And node IDs in rustc. This should help remove our performance problems with SipHash.
- acrichto: Everyone OK with a warning on calls to write due to the IOError?
- pcwalton: Simplicity of not having the extra trait is worth it.
- nmatsakis: Why not just the encoder trait?
- acrichto: It does Write<T> for all these extra primitives. Otherwise, have the writer, encoder, hasher, etc.
- pcwalton: Should Encoder also be Writer?
- acrichto: It's an API that also needs redesigning.
- brson: Any further decisions on this topic?
- acrichto: erickt also brought up that with the Hasher, you have default methods you can override, so you could have a different way to hash pointers as a different way to hash integers. It means there's no way to override the hash for your primitive types. We don't have Write methods for things like pointers...
- pcwalton: We should add one.
- brson: OK. I think we have enough decisions to keep moving. Also, cmr has joined the call. Welcome, cmr!

# Proposal for associated static methods

- nrc: There's an RFC for  a proposal for associated static methods. #12358 - please have a look at it.

# fixed-length vector syntax

- brson: #9879 to reconsider it. I'm proposing we just don't do that and stick with what we have. If we do need a fixed-length string syntax, we'd probably have to use the vector one...
- pcwalton: We may end up removing string types anyway.
- pnkfelix: But string literals would stay?
- pcwalton: They'd be a lang_item
- brson: Sounds great - let's have a great week, Team Rust.
