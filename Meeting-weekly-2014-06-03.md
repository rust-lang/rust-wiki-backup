# Agenda 6/3/2014

* clearing old RFC PR's (brson)
- byte literals (brson) https://github.com/rust-lang/rfcs/pull/69
* Plugins and loadable lints (kmc) https://github.com/rust-lang/rfcs/pull/86 https://github.com/rust-lang/rfcs/pull/89
* Remove markers and replace with a default rule for unused type/lifetime parameters (nrc/nmatsakis) https://github.com/rust-lang/rfcs/pull/12
* by-value-upvars (pnkfelix/pcwalton) https://github.com/mozilla/rust/pull/14620 see also PR 14501
* unsafe pointers (acrichto) https://github.com/rust-lang/rfcs/pull/68
* RFC for attributes on statements and blocks (not match arms) (nrc) https://github.com/rust-lang/rfcs/pull/16
* -C no-split-stack (acrichto) https://github.com/mozilla/rust/pull/14394
* internationalization (acrichto) https://github.com/rust-lang/rfcs/pull/93
* Add on String (acrichto) https://github.com/mozilla/rust/pull/14482
* pub use globs (acrichto) https://github.com/mozilla/rust/pull/14509
* stylistic lints (acrichto)
* remove f128 (acrichto)
* partial_cmp (acrichto) https://github.com/rust-lang/rfcs/pull/100
- size hints Reader Writer (brson) https://github.com/rust-lang/rfcs/pull/46
- homogeneous tuples (nmatsakis) https://github.com/rust-lang/rfcs/pull/104
- libserialize rewrite (erickt) https://github.com/rust-lang/rfcs/pull/22

# Attending

cmr, brson, bjz, zwarich, aturon, nrc, erickt, dherman, lars, niko, felix, luqman, kmc, pcwalton, simon, jdm, azita, jbailey, huon

# Status

- cmr: lexer, grammar
- brson: servo ww, vpc

# Action Items

* (brson) merge https://github.com/rust-lang/rfcs/pull/69
* (acrichto) merge https://github.com/rust-lang/rfcs/pull/86
* (pcwalton) update https://github.com/rust-lang/rfcs/pull/97 with missing content from 77
* (pcwalton/niko) keep brainstorming about captures
* (brson) merge https://github.com/rust-lang/rfcs/pull/97

# RFCs and how we clear them

- brson: Might want to clear out RFCs in reverse chrono order in these meetings. From now on, the agenda will also be combined with some old RFCs, instead of just cherry-picking the easy RFCs. This week, though, we are pretty booked.
- pnkfelix: Can we put the scheduled list of RFCs early so that people can read them?
- brson: Yes. And we will send out the list before the meeting.

# byte literals (brson) https://github.com/rust-lang/rfcs/pull/69

- SimonSapin: There was an issue about this before the RFC process, which I rewrote as an RFC. This is about byte strings and characters. It contains not just UTF8 but any kind of character... the RFC has consensus.
- brson: Any language-level changes, or just a macro?
- SimonSapin: A new type of literal in the languages.
- pcwalton: I'm in favor. i was interested if it could make the Path module easy to use, but with Windows there is more discussion. Even if not Path, it's useful because not everything is UTF8, so having the fallback require putting every single character in single quotes with commas between them is really painful.
- brson: So if it's just a new type of byte string literal?
- nmatsakis: Also byte character literals.
- pcwalton: Yes, having to write `as u8` is also annoying.
- nmatsakis: You had me at literals.
- SimonSapin: We had the byte macro that did this, but it can't be used in macros...
- cmr: They can now!
- nmatsakis: Just because you can expand a macro in a pattern doesn't mean the match arm could actually match it...
- acrichto: Do we remove the bytes! macro?
- SimonSapin: Open question.
- nmataskis: Why keep the macro around?
- erickt: You could encode a byte literal easily with the macro... but I don't think we need it.
- brson: Does the removal of bytes! have to go into the RFC itself?
- acrichto: I'd like to see it removed at the same time, but it's not necessary.
- brson: I will merge this.
- acrichto: backslash escapes same ones supported?
- SimonSapin: \u and \U are not supported...
- acrichto: Can still do \n \t, etc.?
- SimonSapin: Yes.
- acrichto: Raw byte literal at some point?
- SimonSapin: Open question. If we have this, there's no reason not to add it, probably at the same time.
- brson: Why no unicode escape sequences? Don't they translate to a sequence of bytes?
- SimonSapin: You have to pick an encoding for them to get a sequence of bytes...
- brson: Done.
- erickt: The type of a byte literal will be a byte slice/vector?
- SimonSapin: Slice of u8

# Plugins and loadable lints
https://github.com/rust-lang/rfcs/pull/86
https://github.com/rust-lang/rfcs/pull/89

- kmc: Just plugins today, because lints are still in flux. The change generalizes the registrar. It's implemented, and the only point of contention is that syntax extensions depend on librustc. Unfortunate, but the practical consequences seem low.
- brson: For the macro registrar? Any benefit to having macro and plugin registrars separately to break the dependency?
- kmc: Yes, but lots of duplicated plumbing, then. Currently, external users of libsyntax can't load syntax extension crates without metadata parsing from librustc anyway. You have to parse metadata to load the macros... so the dependency was already implicitly there.
- brson: By expanding the API surface, do macros become even less specificiable?
- acrichto: Different because there you're manually opting in to the entire compiler vs. being required to link against the compiler.
- kmc: Could add a lint. But seems like not a big deal...
- dherman: Will macros become powerful enough not to use syntax extensions / low-level plugins?
- kmc: No. I have one that opens up the whatwg JSON file and outputs rust code...
- dherman: CHALLENGE ACCEPTED! We want programmatic macros, right?
- pcwalton: I believe what we want is for the registrar to not be exposed, ever, since it's not a very nice API. Requires programmatically adding your macros to the symbol table, which is not what we want people doing. Ideally, the supported way is via some other declarative mechanism. So a symbol at a location has expansion code at blah blah blah, but there's no explicit symbol table manipulation in this code. I don't care if the API for the registrar gets more complicated because it's a bit of a dead-end anyway w.r.t. macros. Shouldn't have to add anything to the symbol table manually.
- dherman: The core thing about macros is that they're scoped, so they're in a module and you export them, etc.
- pcwalton: Declarative instead of imperative mutation of symbol table.
- kmc: Agreed. Maybe v1 is metadata... the registrar is definitely a low-level API.
- dherman: A plug-in architecture for a compiler is a fine thing, but should mainly be for implementation of the compiler, not as a general language feature. I understand it's a long way off before the macro system is there, and this is the only recourse people have, but ideally we would move away from that.
- kmc: Yes.
- erickt: If the only dependency on librustc is for general macro plugins, can the metadata parser be factored out?
- kmc: Yes, that and the stuff that can dlopen a crate (the registrar) and then we'd change the registry to have a bit more trait magic per the discussion in the RFC to avoid having to load crates that define the types. But that's a lot of work right now and I'd like to get a v1 and some plugins together.
- pcwalton: Wanted to move metadata to a separate crate for years, but the metadata info is a core part of librustc.
- nmatsakis: A remotely stable internal compiler API is a huge endeavor.
- pcwalton: I'd just like to see something more declarative than worry about the API for the registrar.
- brson: It is exposed in several critical crates, though, and we may end up committed to this...
- pcwalton: Don't see any alternative than to say the registrar is unstable.
- kmc: We use it for logging, but we control the implementation of that crate.
- brson: It may end up defacto-stable. 
- pcwalton: One problem with the macro registrar is that the symbol table it uses is the global symbol table, and fixing that is a lot of work. Either continue with what we have or have to fix all of hygiene...
- brson: Metaprogramming is a bit of a mess right now. But are we making it worse?
- kmc: It's no less declarative than the current stuff.
- brson: I does increase the risk of additional things we need to be back-compat with.
- kmc: Anyone writing a syntax extension is touching really unstable internal compiler stuff. The plugin registrar does not make things much worse.
- pcwalton: I agree with brson that this makes it worse, but it's already so bad that it's not measurably worse. If we have to have a defacto-stable compiler API, we're already doomed. We need to focus our energy on making that not come to pass rather than try to mitigate the damage if it does come to pass...
- nmatsakis: I'm OK with this.
- acrichto: I don't like it, but let's merge...
- brson: Let's do it. Who will merge it?
- acrichto: Me.

# Remove markers and replace with a default rule for unused type/lifetime parameters (nrc/nmatsakis) https://github.com/rust-lang/rfcs/pull/12

- nrc: Been trying to go through some of the older RFCs. This is one of them, and it's niko's and we should just merge it. Nothing holding us back on it.
- nmatsakis: I agree! :-)
- brson: What is this RFC?
- nmatsakis: Question is what should our behavior be w.r.t. unused type/lifetime parameters. Current is a mess. Two options: 1) (this RFC) is to behave as if the type parameter were used to store data. So, it's covariant. Option 2) is make this an error and you have to use a marker type to get the variance. My justification for #1 is that they're equally expressive in practice, but you really always want the former. Usually, it comes up when somebody is making a funny type implemented in a goofy way that needs to act like e.g., a pointer to T.
- pcwalton: Most of the objections are in regards to GADT. Which Rust does not have.
- nmatsakis: I do not yet fully understand the GADT and other objections related to features Rust does not have. The question is: is there a role for bivariance? If we go that route, we could make a marker for that.
- pcwalton: The problem with markers is that you have to write covariant and contravariant, which seems too pointy-headed..
- pnkfelix: The alternative of a fake type to express what you want with a lifetime argument seems just as pointy-headed.
- nmatsakis: The one instance would be the Unsafe type (it's invariant w.r.t. T in order to get interior mutability). But that's built into the language...
- pnkfelix: What if you want contravariance?
- nmatsakis: Nobody wants that! 
- cmr: Can we leave in the marker types and change the default?
- nmatsakis: Could make it an error. Right now, it falls back silently to a poor default. Or an error is fine. I haven't pushed this RFC more because I'm somewhat torn, based on how often markers have to be used.
- dherman: There's a false dichotomy between inference and using co/contra/in variant. C# used in and out, which are much more approachable. I'm not injecting bikeshedding on the names, but perhaps we could use names that aren't pointy-headed for the markers.
- nmatsakis: Yes. True.
- pnkfelix: I vote for the error if you don't use the type parameter so that people say what they wanted, because just guessing "covariant" seems bad.
- nmatsakis: I think you've convinced me.
- pcwalton: I disagree. I think we should do what you almost always want, and you have to do this all the time with custom smart pointers.
- nmatsakis: Let's table this. I'd like to gather some statistics.

# by-value-upvars (pnkfelix/pcwalton) https://github.com/mozilla/rust/pull/14620 see also PR 14501

- pnkfelix: I broke up pcwalton's big commit into a bunch. This is switching to the by-value upvars rules for closures. I'm not necessarily against this change, but it's a big change and we should all take a look at and understand this. Acrichto has some comments on some places where there may be issues with code sharing.
- acrichto: My comments are all very minor, though.
- pnkfelix: Just wanted to bring it up so that everyone's aware of it. I was considering not trying to land this as-is today, but keep supporting byref upvars and guard it so that it's deprecated. So there's a feature gate for the near term to keep your code compiling.
- pcwalton: Strongly opposed. Everything we mean to kill before 1.0, we should not make concessions to old features we have deprecated. We have no enterprise customers here.
- brson: Every time we've made back-compat efforts, people have appreciated it...
- pcwalton: This is all tied up in unboxed closures. We can have unboxed closures and have byref upvars (capture clauses).
- brson: So you want to see the byval thing and see if we need to add capture clauses?
- pcwalton: yes. I don't want to keep two wildly different types of closures in the language.
- brson: I thought it wasn't going to have a lot of impact, but it looks like there's a local declared everywhere...
- pcwalton: Yes, just 10%, but there are a LOT of closures in rustc. 
- acrichto: It's every person who ever wrote a benchmarking function now has to capture something, though!
- pcwalton: The benchmarking API is not good.
- nmatsakis: This is why we had two closure creation forms originally... byval and byref.
- pcwalton: We can bikeshed our syntactic forms for byref, whether it's capture clauses, a different closure expression, etc. 
- nmatsakis: I just bring it up because if we add it, I'd hate to have done all this work and then undo it.
- pcwalton: I do think we shouldn't keep two kinds of closures around because that's a lot of maintenance burden to the compiler. It also doesn't encourage people to move to unboxed closures, since they will just add the flag to keep their code compiling, and people just keep using boxed closures. 
- nmataskis: Well, if we did add a capture clause and the default is byref, then we did a lot of changes (and other people did too) for no point. I agree that straddling two worlds has always caused us pain. But do we have any second thoughts on this change now that we have an actual diff? Reading the code changes, there are a lot but it's hard to get a sense.
- pcwalton: I wasn't aware that byref as a default was an option...
- nmatsakis: That was always the plan; it was what we did until now.
- pcwalton: byvalue is more common... I thought that byvalue by default was going to be it with maybe byref added.
- nmatsakis: pipes byref and proc byvalue was what I thought.
- pcwalton: I assumed proc would go away.
- nmatsakis: The plan was, yes, we might be able to live without proc and it's better to have one simpler construct with a little adaptation rather than two constructs. The thing about the statistics is that I agree it's the wrong default sort-of except how often do closures escape outside the stack? Because that's when it matters. Clearly all of these closures don't. I think I'm still behind theplan, though.
- brson: Another big change here, though. Can we make a post on the mailing list about what we're thinking and how we'd like to see how things turn out? Is anybody really concerned about going forward with this right now?
- pcwalton: I can send something to the mailing list.
- acrichto: I have issues with byval captures by default, because every line of diff in this patch looks like a step backwards in readability.
- pcwalton: We have PRs for byvalue upvars, closure literal syntax, sugar for function mut trait, and finally for allowing you to call the FunMut trait with parens. I haven't done all three traits yet, just FnMut.
- brson: How many RFCs?
- pcwalton: Two. Does anyone thing unboxed closures are a bad idea?
- acrichto: Seems a bit hasty, but not a bad idea.
- nmatsakis: I think what alex said is targeted; it's hard to say this PR is a step forward in usability.
- pcwalton: When you say unboxed closures, people look at the upvars thing, but that's just about upvars moving by value or not and related to the changes. But we could do byvalue with unboxed closures...
- nmatsakis: I don't think anyone is objecting to the feature, just the specific design.
- pcwalton: pnkfelix was saying to keep boxed closures in, but that's a separate feature.
- pnkfelix: Please continue.
- nmatsakis: I belive he was saying we could keep byref semantics in.
- pcwalton: We'd need to have a separate syntax to keep byvalue in. Hadn't considered proc as a potential syntax for capture clauses. I'd like to merge my unboxed closures PRs. Is that OK?
- brson: Separate?
- pcwalton: Yes. The three PRs are all blocked on RFCs.
- pnkfelix: You can land the other PRs without this one?
- pcwalton: Yes. They create a new syntax alongside the existing one.
- pnkfelix: Okay.
- nmatsakis: We have to decide quickly, because it's going to bitrot.
- pcwalton: I vote that we accept & merge unboxed closures RFC with the caveat that we may add capture clauses later. One possibility for captures is byvar & byref stuff handled in it. But separate RFC. I think unboxed closures are a good idea and we should merge it.
- nmatsakis: Okay. Sounds reasonable.
- brson: What's the plan again?
- nmatsakis: I'm reading the notes :-)
- pcwalton: Unboxed closures do override the other one in a way, as it tuples arguments instead of infinite traits
- nmatsakis: Which ones?
- brson: 77 & 97. I believe the infinite traits portion of the RFC was laying out a design we would not use, correct?
- pcwalton: Yes, that's the RFC I submitted with the details around unboxed closures. 
- nmatsakis: Lots of missing stuff... but Fn, FnMut, and... the other trait? Yeah.
- brson: This is a very big topic.
- pcwalton: I don't want my PRs to just sit there.
- nmatsakis: What's in 97 seems like it meets our criteria of merging the RFC. 
- pcwalton: Let's merge 97.
- pnkfelix: But 97 says it builds on 77. Should we try to rewrite this?
- nmatsakis: What's missing from 97? 
- pcwalton: Just that unboxed closures implement a struct that is not named.
- nmatsakis: Copy-paste and call it a day.
- brson: Do that and we can merge it. Then we can merge two of your PRs to implement that RFC? The only contentious one of your PRs is the byval one. Are we ready for that one, too?
- acrichto: it's all feature-gated, isn't it?
- brson: No. We're talking about the one that you were uncomfortable, acrichto. I think we're going to merge everything but that.
- acrichto: I won't block this change, but I also won't support it.
- nrc: According to the RFC, we will have capture clauses where we can specify stuff taken by reference. If we going to implement that soon, would we undo changes in this PR?
- pcwalton: Yes. Maybe we should decide on capture clauses?
- brson: Seems almost like we're going in a loop
- pcwalton: Beginning to wonder if capture clauses are required
- brson: Prefer envirnoment
- nmatsakis: I do feel like that last PR might be better to see if we can come up with a syntax that handles it all.
- pcwalton: I care more about the other PRs anyway.
- brson: We have consensus.
- pcwalton: We will try to come up with a syntax for capture clauses later.
- brson: pcwalton will update 97 with missing data and i will merge it. Take everything he has in the queue except byval captures to merge, and he will bikeshed capture clauses.
- cmr: For function traits, can you implement them on anything?
- pcwalton: Yes. It's just a trait.
- cmr: Before that, you couldn't just implement callable on anything...
- nmatsakis: It was part of the RFC.
- brson: Out of time! Thanks, all.







