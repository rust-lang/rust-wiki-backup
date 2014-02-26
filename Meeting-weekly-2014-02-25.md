# Agenda 02/25/2014

* weak extern fns (acrichto) https://github.com/mozilla/rust/pull/11978
* hex floats (acrichto) https://github.com/mozilla/rust/pull/12323
* channel names (brson) https://github.com/mozilla/rust/issues/11765
* unsafe ptrs, *mut (brson) https://github.com/mozilla/rust/issues/7362
* relative ref // req leading `::` in use (pnkfelix) https://github.com/mozilla/rust/issues/10910
* bors capacity (brson)
* contract updates
* TotalEq/TotalOrd (pcwalton)

# Attending

- pcwalton, acrichto, cmr, huonw, dherman, brson, nmatsakis, pnkfelix, larsberg, jack, jdm

# Status

- acrichto: windows file fixes, SVH, rustdoc, if_ok/try, remove std::run, LLVM arm patches
- nrc: type variance, struct inheritance, a couple of small things, last layout bug (css images - list-style-image)
- brson: docs, stability, atomics, workweek
- nmatsakis: workweek prep

# Friend of the tree

- brson: Erick Tryzelaar (erickt). Since May 2011, he has done countless things. One was the serialization crate. He organizes the local Rust meetups. He made a big effort to fix our Hash trait and landed that this week after a month of issue talks.

# Notes

- brson: We have a doc contract with Sam Wright. He'll be working on the tutorial. He's been working with Niko and I, but you will be seeing him in the community soon.
- brson: Michael Woerister will be on contract working on debug info features. He did it before in GSOC last summer. 
- brson: bors is COMPLETELY overloaded right now. We have 44 things in the queue. We're in the process where we start to have to roll multiple pull requests into one.
- pcwalton: Biggest problem is android?
- brson: No, android is fine. It's the fastest.
- acrichto: The 32-bit bot is sometimes on our slow bot. But it's +/- 10 minutes. THe biggest problem is we have a 15 minute bors frequency between our PRs that we might be able to fix...

# TotalEq/TotalOrd

- pcwalton: There's a PR to revamp this stuff. I think it's an improvement, but it still has 4 traits. I'd like TotalEq and TotalOrd to die, but I want their replacements not to be Partial. I want just Eq and Ord. Everything else is really complicated. I propose merging Eq/TotalEq and Ord/TotalOrd. And, Eq, should have a method `equals()` that corresponds to the total one (with no default). And another method `eq()` should be defined in terms of `equals()`. Ord should have an undefined `cmp`, which returns an order. Then, four methods for overloaded lt, le, gt, and ge are defaults in terms of cmp. Almost never will you override the eq, ne, le, lt, gt, and ge. The only ones that will are f32 and f64, which will do it for IEEE754 ops. For float32 & float64, equals will be implemented as bitwise equality. cmp will be defined as "cast floats to integer bitwise and do that comparison." Why? That's what's useful for making floats keys in hash tables or binary trees. Those two should use equals and cmp. Everything else, including sorting, should use lt, le, gt, and ge, and should use them through the operators. This provides you with: only two traits; only two methods to implement; hides the complexity of partial equality. It does the right thing for sorting e.g. an array of floats and is IEEE754-compliant, so NaNs do what you expect, but you can still put NaN in your hashtables. Hashing and binary trees are still very fast per sunfish's guidance on how people do this.
- nmatsakis: Would deriving work? If you derive either of these traits, will (e.g.) `lt()` be derived in terms of `lt()` or `cmp()`?
- pcwalton: I'm not sure. It means that comparing a tuple of two floats... you want the IEEE comparison, right? I think deriving should define lt in terms of lt.
- acrichto: Nobody creates trait objects out of Ords...
- nmatsakis: I had a summary of the three options here from IRC conversations. The problem is the difference between IEEE's behavior on floats and mathematical ordering. Three choices: 1. support only true ordering (but that's slow and surprising). 2. a middle ground, where you have both partial and full ordering and they don't necessarily behave the same. Operators do partial and equals does full. 3. distinction between partial and full, but enforce that if you have the full, partial supports the same result. In practice, you'd get two separate sets of traits and the floats don't support the total ordering. But, you could have a newtype'd float that supports it. There are then a bunch of variations on those three options.
- dherman: I don't claim to have the full intricacies, but I like patrick's capture of the space. It avoids the proliferation of type classes and has the maximal efficiency. I don't think I'm informed enough to have an official weigh-in on it, but pcwalton's proposal sounds really good and solves those constraints.
- pnkfelix: What are the laws that eq, ne, lt, and le follow? 
- pcwalton: None.
- pnkfelix: Writing generic code over Ord means that it's not generic if there aren't any laws...
- pcwalton: This is an IEEE754 problem. There are only two fast things to do on floats. IEEE754 less-than and casting bits to an integer and comparing those integers (for equality). If you don't have fast comparison, you're going to have a hard time with sorting an array of floats.
- pnkfelix: The question is whether it's OK for f32/f64 to not support Ord?
- pcwalton: Can't sort them.
- nmatsakis: If you have four traits, then sort can require only partial order...
- pcwalton: But then four traits!
- pnkfelix: I like separating the laws from the behavior...
- pcwalton: Four traits is a burden.
- acrichto: We could have the traits in libnum, but not the standard library.
- nmatsakis: I like pcwalton's design, but we're trading simplicity in traits for some complexity in the methods on those traits. They will behave differently on certain types. And, if you have a generic parameter of type T that doesn't support lt the right way, it just won't work in a binary tree, for example.
- pcwalton: I think binary trees will be using cmp anyways. But still, it's a possible trap for people implementing containers. But that's a smaller number of people than trait implementers. I don't think using Haskell as the baseline is a good idea because they have different optimizations that implement different ordering semantics.
- nmatsakis: The counter-proposal is Ord and Eq. They have cmp and equals, as per pcwalton. Separate traits for eq operator lt operator, etc. The only ones grouped together today are some of them...
- dherman: What do you mean laws? THe ones automatically enforced? Or the ones we expect the implementer to enforce?
- pnkfelix: If I write code that's generic over those operators, what are the guarantees that I get using those things?
- dherman: The ones that are supposed to be provided by the trait? Also, pcwalton, you must mean there's *no* law that holds...
- pcwalton: I don't know how to summarize them w.r.t. IEE...
- dherman: Isn't there a partial order? 
- nmatsakis: `a<b => b>a`
- dherman: Just NaN != NaN, right?
- pcwalton: and positive/negative zero.
- dherman: We don't have to enumerate. I thought that there's a partial order, at least...
- nmatsakis: I'd like to move the operators into their own traits and have Ord and Eq just be the one method...
- pcwalton: I'm strongly opposed to all the traits.
- nmatsakis: I'd like to separate the equality from ordering traits.
- acrichto: What if you take the lt bound when you meant ot take the ord bound?
- nmatsakis: Not bulletproof, but it's more evident which trait you meant to choose.
- brson: I agree with Niko's concerns about naming. I'd want them to be more clear. Remembering that equals is total and cmp is not is hard. Second, I'm not clear what sacrifices we make w.r.t. the IEEE total ordering we sacrifice with the integer cast.
- pcwalton: You know that -1 < -.025... but not 
- nmatsakis: You could have multiple NaNs that are not equal
- pnkfelix: Would they be adjacent? Or anywhere in the order?
- nmatsakis: Anywhere.
- acrichto: It provides a total ordering, but not necessarily the total ordering.
- nmatsakis: It's it weird that two NaNs might not land on the same key?
- pcwalton: I hate imposing a huge burden on all users just because of floats.
- nmatsakis: But having everything implement the operators is a burden... telling people not to use the operators is a burden
- pcwalton: If you want a partial order, you use the ordering. Also, think of other languages: JavaScript does not need a total ordering.
- pnkfelix: Why do binary trees require a total ordering?
- pcwalton: A binary tree needs a total ordering; sorting only needs a partial ordering.
- nmatsakis: Sorting doesn't need to compare against equality, whereas a binary tree does.
- pnkfelix: weird to walk the leaves of a binary tree and have them not be equal to a sorted array...
- nmatsakis: Probably right that this isn't a big issue in practice, pcwalton. Depends how you feel about having more traits and more operators. We already have a lot (bitwise and and bitwise or). Maybe one big trait (Ops?). Doesn't seem crazy to have lte unimplementable but lt is...
- pcwalton: Never that case.
- brson: What are other thoughts on this PR?
- pcwalton: I think this PR mainly a rename...
- pnkfelix: No, it keeps the deriving in place! not just a rename.
- nmatsakis: Can either have partial XOR total ordering on a type, but never both and they can't diverge. It's different.
- pcwalton: You really need lt on floats to be partial, so I think it's a non-starter.
- pnkfelix: A newtype'd wrapper could implement that.
- pcwalton: And you'd have to do that whenever you want to put a float in a hashtable? And we still have 4 traits?
- pnkfelix: Just factors out operators out into separate traits. SImilar to what we do today. My take is that 4 traits won't be that bad because people won't tend to use the partial one anyway.
- brson: Having the correct ones have a simple name is better....
- pcwalton: I think there's diagreement about whether multiple traits is bad.
- dherman: It's bad to have multiple traits when it's hard for people to choose which to use. Or if you choose on and then can't use other code because it relied on the other trait. So, let's say that TotalOrd and TotalEq are the common case. In pnkfelix's case, when is Partial the right case? Only floats?
- pcwalton: Partial is almost all comparisons (e.g., sorting). The question is: when do you need total? In associative containers, basically. Everything else is partial.
- nmatsakis: Starting to persuade me. In the vast majority of generic code, what is the right choice?
- pcwalton: Some generic code wants partial... almost all. 
- nmatsakis: Would it be better to call the methods totaleq and totalord? If it's so rare that we use them. We can call the others partial, sincet hey'll have operators anyway.
- pcwalton: Feels a bit academic...
- nmatsakis: If possible, the operators should not be simply called lt. If you just see lt and cmp, you'll assume they do the same thing.
- pcwalton: Given the type signature of cmp, it has to be total. Maybe that's too subtle?
- nmatsakis: I guess not that many people are calling it, if you're correct.
- brson: Are we going to come to agreement on this today?
- nmatsakis: I'm close to giving in...
- pnkfelix: There's pushback against having the operators in their own traits....
- dherman: I'm concerned about consistency, if the other operators are in their own traits!
- pcwalton: Unless eq, ne, gt, ge, all in separate traits, we're inconsistent. That's a non-starter unless maybe we have trait inheritance.
- nmatsakis: There's no shorthand for multiple traits right now. Maybe could have an impl for all types T...
- pnkfelix: Wouldn't deriving get it for you?
- nmatsakis: Doesn't seem bad to have Ops... but this can wait another week. We've been discussing this for years, I believe.
- dherman: Should resolve by the end of the workweek.
- nmatsakis: pcwalton, can you write that up as an RFC?

# weak extern fns

- acrichto: We do not have any other kinds of linkage other than the default. This PR adds support for weak linkage to external things. Not Rust fns. The type system doesn't allow this because the typesystem assumes it's not null but it can be because of weak linkage. Should the type of a weak function be Option<fn>?
- nmatsakis: Option would be nice...
- pnkfelix: Seems like the most logical thing.
- acrichto: static SOMETHING: type has value Option<type>? Also, does it mean the address of the thing is the type instead of the Option?
- pknfelix: Is it only functions that can be linked weakly, or all symbols?
- acrichto: All symbols. Could restrict it to a smaller set.  THe idea is that we call a function on linux if it's available (use dlopen + dlsym, which is... interesting).
- nmatsakis: Doesn't seem that weird if its type is option. Seems like if there was a weak keyword for `weak static i : ...` maybe not so weird. Or maybe require Option?
- brson: Could be a static variable that's an Option<fn>. Enforce that weak = Option type.
- pnkfelix: So weak is only for statics? Aha, yeah, that would be fine.
- brson: But rustc doesn't know about Option right now. Doing it for just that is bad.
- nmatsakis: True!
- acrichto: Also not sure we can use Option. A weak linkage is to the symbol... so it's not actually an Option. It's actually a pointer. 
- nmatsakis: What did that mean?
- acrichto: Weak linkage to a function pointer is different from weak linkage to a function.
- brson: We don't need this, do we? We already have dlsym, right? Maybe we should keep thinking about it... seems hard...
- pcwalton: Not hard; just complicates the type system.
- nmatsakis: That's what makes it hard.
- acrichto: Can't assume it's a u8 because it assume &u8 can't be null.
- cmr: (Are they actually null, or) Do they explode if you (Call/) dereference them?
- pcwalton: Wait, if it's a *u8, of course it can be null.
- acrichto: How do you get the *u8?
- nmatsakis: the linker doesn't know the type, right? It just does it by name?
- pcwalton: yes.
- nmatsakis: So you can have weak symbols, but it just has to have * type. 
- acrichto: That could work. Tiny bit of codegen work, but could work... I like that.

# channel names

- brson: HUGE bikeshed in issue tracker. On the verge of redesigning this API. I think we have consensus... but not everybody's happy.
- acrichto: Chan::new() returns a tuple. We also have the names Chan and Port, which confuse the concept of a channel with the channel and port types. Idea is to make the constructor more idiomatic and the names to be crystal-clear. The "accepted version" is a function called `channel` in the prelude. I think that will turn out well. The two halves are all over the place. Transmitter/reciever, sender/receiver, all sorts of other stuff. And if we want to add synchronous channels, we need to have some design room for that (including new names on sending/receiving side. I think we're settling on "sender" and "Receiver" with "syncsender." I did a sed replacement and it looks terrible, though.
- nmatsakis: Why?
- acrichto: It's really nice having four letters. Sender/receiver seem wordy and out of place. It was often...
- dherman: Reader/writer has the same number - was that a problem?
- acrichto: Those are in IO. I think we should keep it there.
- pnkfelix: Local renaming for people who like four-letter words is not good enough?
- dherman: I share the column-obsessiveness, but also remember that is balanced against the port/chan problem of not having no obvious mnemonic. To this day, I have to stop and think about it and I've known the terminology for years. If it's down to those two, what's the lesser of two evils? In mine, columns are not as important as being memorable.
- nmatsakis: sndr, rcvr
- cmr: That's nasty!
- dherman: We're all smiling sarcasticly :-)
- acrichto: Also, what locally? tx,rx is nice. I use p,c.
- brson: sndr?
- acrichto: That's horrible...
- pcwalton: car, cdr? or fst, snd?
- dherman: good example of The Worst Possible!
- brson: Will anybody have a fit with Sender/Receiver? Then let's do it.

#  relative ref // req leading `::` in use

- pnkfelix: I used to be confused by use declarations using absolute paths and everywhere was relative. I was suggesting that we might change the rules and having leading colons on use declarations. More recently, glaborela? recommended having relative paths in use declarations. I've gotten used to things, but I remember it was bad when I was a newcomer
- cmr: Comes up in IRC all the time
- pcwalton: I want to be able to type std:: and don't see how we preserve that? Have to do ::std::, and that's a lot of weight...
- pnkfelix: No, if we switch to relative and the prelude include use::std, then it's fine...
- brson: that's only for std
- dherman: How do you force relative?
- pcwalton: self::
- dherman: The lexical syntax might make a difference, but in the JS world we default to absolute. If you want relative you use `./` (instead of self). 
- pcwalton: Just using . instead of self...
- dherman: I'm just saying there's a world where absolute seems to work
- pnkfelix: What's the analogy to use?
- nmatsakis: When we move to absolute paths from relative, the code got much easier to read.
- pcwalton: The search also started at your ancestory hierarchy, and interacted with globs, etc. It was really bad.
- pnkfelix: I don't care as much anymore, but it comes up in IRC all the time and I wish we had something easier for users.
- brson: We may be stuck where we are in the design space.
- pcwalton: I don't see anything we could do that doesn't make this worse
- brson: We probably just need better compiler errors.
- nmatsakis: Maybe the problem is you can't use an absolute path in the source without the ::? In Java, you also you can paste from the import into the code.
- brson: I often put the :: stuff in the imports because it's so ugly.
- pcwalton: What if every time you used :: it was absolute? Even in the code? 
- nmatsakis: breaks `use std::vec; ..... vec::`
- pcwalton: Right!
- pnkfelix: Hrm, what was the proposal? Try relative then fall back?
- nmatsakis: That's what Java does.
- pcwalton: Java is more of a package naming thing...
- pnkfelix: I'll close that ticket.
