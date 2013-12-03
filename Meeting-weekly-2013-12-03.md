# Agenda 12/03/2013

- Result api redesign (acrichto) https://github.com/mozilla/rust/pull/10364
- str::from_utf8 (acrichto) https://github.com/mozilla/rust/pull/10701
- "enum mod" (acrichto) https://github.com/mozilla/rust/issues/10090
- libcxxabi/libunwind to drop libstdc++ (acrichto)
- rustpkg (jack)
- "box" (pcwalton)

# Status

- acrichto - static linking landing + fallout, LTO (last night), rewriting std::comm
- brson - android bots (having problems)
- nmatsakis - DST planning / formalization (getting somewhere), borrinwg & in & mut, closures
- pnkfelix - codemesh talk, DST discussion, GC
- pcwalton - figuring out "new" + burning through lang-backcompat

## placement new

- pcwalton: There was a long conversation about this over the holiday on the mailing list. There were concerns around `new`, mainly that it conflicts with constructors (which we knew), but also that when used with a value is strange (e.g., `new 10`). At odds with C++ as well, since there's  a value instead of a type. Lots of proposals here. Most support was for `box` instead of `new`. Has the advantage that it's got the right intuition "it's the boxing operator." Second, people can still use new for constructors. Third, reads better with values. `let x = box 3` reads nicely. It's nice and short, a full word (instead of `alloc`). Opinions?
- pnkfelix: Another problem with `new` is that the destructuring form on the LHS of an = is ugly. Box does not have that problem.
- pcwalton: Good point. Could overload with *, but with box, we don't need the hack.
- nmatsakis: Box kind of becomes that overloaded *, but it's more in line with using the creation syntax on the destructuring / matching side.
- dherman: Yes, box is consistent.
- nmatsakis: I'm in favor of this proposal.
- dherman: box unqualified is always a unique type? 
- pcwalton: Yes.
- dherman: And qualified, it's whatever the placement new type constructor specifies?
- nmatsakis: Could have an explicit value for ~. Could try to use the result type to try to make it a little shorter.
- pcwalton: We usually don't try to invoke user methods based on type inference.
- nmatsakis: Operators, though... I'm not sure I like my idea, but wanted to throw it out.
- pcwalton: May result in a big annotation burden.
- acrichto: What's the type signature of a unique pointer? Still have a ~? Or `box`? Or *?
- pcwalton: I prefer *, but separate conversation that would probably take up the whole meeting.
- nmatsakis: Plan is ~ for now.
- brson: Any objections to box?
- nmatsakis: Not renaming new is nice.
- jack: No strong feelings. The placement part was weird. Something in parens followed by something else in parens was a bit confusing. Maybe `in` for place?
- nmatsakis: Too many words. Evaluation order is also reversed on the placement stuff.
- acrichto: Angle brackets?
- nmatsakis: `box(GC)(2)` doesn't seem too confusing...
- jack: Yeah, just confusing when using foo instead of GC
- dherman: Using `::`` for type parameterization, could use it similarly for placement?
- nmatsakis: It's not a type, so that's kinda misleading. Another reason to argue against my inference idea. The type-based one would work with GC and the value-based one would work with arenas.
- pnkfelix: Do we have other noun keywords?
- nmatsakis: It's a verb.
- pnkfelix: I know it's used as a verb. But it overlaps with noun usage.
- dherman: Are we getting value out of the noun/verb ambiguity? In pattern match, it's a noun; in the operator, it's a verb.
- nmatsakis: That's proably why `new` doesn't work on the LHS!
- dherman: Before everybody silently agrees, `new` keeps 80% of what people already know "this allocates." When it's a totally new keyword, you're basically saying, "you have no idea what this code does - go to the manual." That's a huge language cost we should be aware of. 
- pnkfelix: I expect it to be used by library types.
- dherman: I just mean people coming to the language. The on-boarding experience for first-time users of Rust. If they see a language with familiar keywords, they understand the tradition, basic ideas, etc. When you pick new keywords, you leave people lost and many will give up.
- pnkfelix: I don't know how often it will come up. Will people see `box`?
- nmatsakis: Only if we replace ~ with `box`. 
- dherman: So, wait, we don't replace ~ the operator, or ~ the type?
- nmatsakis: If we keep the operator, it delays when you hit box. 
- dherman: I thought acrichto was asking about the type syntax changes with ~. But that nobody suggested keeping the operator ~.
- nmatsakis: Right, I just brought it up, but I don't really like keeping the ~ operator.
- pnkfelix: People in the chat rooms and on mailing list have said we're not supposed to be using ~ much.
- brson: That seems strange. It's like saying, "you're not supposed to be allocating," which is not correct.
- pnkfelix: The ~ was already something new, then. Is `box` scarier than ~?
- dherman: And that's why we were considering `new`! On one side (Dave's), if you share 80% of the semantics, use the old syntax because people are good at handling the mismatch in semantics. The opposite side of the argument - to not use similar syntax for different ideas - is not realistic about how people use languages. Most people who use them do not actually understand the semantics. Worrying about people understanding subtelties between C#, Java, C++, Rust's, etc. `new` is not worth it. People gain more by seeing the similar syntax. That said, we don't have to do `new`, including for pattern matching and conflicting with the constructor pattern, `new 10` is goofy, etc. But the "different semantics than Java" is a bad argument.
- nmatsakis: I agree. The interesting question is are we so far off in semantics that it makes it worth distinguishing?
- dherman: I'm more worried that people will look at our syntax and be totally confused and think the language is just weird. `new 1` is not that terrible.
- acrichto: The first time I saw ~, etc. was in the context of strings & vectors. What's the plan there? Am I going to have to write `box "str"` to get a ~str?
- nmatsakis: Yes. A string constant is a static string right now, not owned. But for vector, you'd write `box [1,2,3]`. I guess ~str is more common. A ~[], though?
- acrichto: A vector of elements is common. But an empty one to push onto with `box []` is weird...
- nmatsakis: vector::new()
- acrichto: &~'key for using a hashmap the first time in a JSON hashmap. If it's &box`key... hrm. That could be a rough experience for first-time users.
- nmatsakis: Strings/vectors questions also interacts with DSTs.
- brson: So, do we have a decision on `new` keyword today?
- nmatsakis: let's have a decision. I vote in favor of `box`.
- dherman: I feel like it would be nice to write up a really fleshed-up argument for this instead of just design-by-conversation.
- nmatsakis: Maybe summarize the arguments?
- dherman: At least, it'll be useful for future bikesheds to point people who want to talk about this again. Having a single document with all the issues will cut off the conversation.
- brson: Who's going to write that up?
- everybody: ...
- brson: I'll try to summarize it
- jack: Do you want to start a RFC-ish process?
- dherman: What would it be?
- jack: Documents are numbered. Has sections, etc.
- brson: Every time we adopt strict process, we stop doing it.
- dherman: Lots of social engineering then. Is it for the team, the community, who wants to be Chief Curator Of Cat Herding.
- nmatsakis: Later.

## rustpkg

- jack: Been hacking on the rust linker and build system to improve servo's build lately. Basically, with a few small changes to Rust, we can have great integration with other tools. I've been thinking maybe we should hand rustpkg to the community. There's a lot of work to make it actually useful; document the external build tooling integration and maybe not take rustpkg as a 1.0 blocker on ourselves.
- nmatsakis: Nervous. I think there's a lack of design understanding of this space on the core team. I'd like to see how build should work, what's in/out, etc.
- jack: I can document what I've been doing. I've used tup and cmake and regular make. I have examples of small crates, crates with c deps, etc. with all of those. I can write that up in a series of blog posts. The rustpkg design is harder, though, because there were some goals and some docs, and it's all sort of in-progress stuff.
- pnkfelix: I was under the impression some members were trying to adopt rustpkg.
- jack: bjz jumped on it first, but it didn't quite work with his examples directory layout, etc. And he's very enthusiastic.
- dherman: Bigger conversation here for the team.
- jack: It would be nice to have the use cases laid out. So does glfw from bjz (e.g., its example programs). Also, some of our constraints - system libraries, etc. The rustpkg design was to put stuff in /usr/lib. gopkg and npm, and pip tend to install in little silos to avoid interacting with the rest of the system. 
- nmatsakis: I agree with use cases. We have to make a call re: 1.0, as patrick says. My separate concern is that we should have some sort of design for this so that it doesn't grow into something we don't want.
- dherman: I agree that we can put off rustpkg from 1.0. We're all saturated. I agree that rustpkg needs an owner, design, etc. and can't just be handed to the community.
- jack: Already growing organically - there's a big PR on it.
- dherman: Probably OK.
- larsberg: Agree with pcwalton that separating build & pkg manage is good. I'd like to table make and just do package managment, personally, but whatever team says.
- dherman: That's what I've heard as well. Not tackling build would be very good. jack may be right, though, that there are some issues with build that we have that others don't. We'll have some other meetings on where to go from here.

## result API redesign

- acrichto: Changing things to Option instead from Result. But doing this change just removes one method from the Result API. So, it's basically saying we don't add the Option methods to result and instead return an Option if you want to get the fancy support.
- brson: Are you saying this doesn't reduce the Result API surface?
- acrichto: It's already really small. People just keep wanting to grow Result, and this says to have two methods to return options.
- nmatsakis: and_then we can't convert, right?
- acrichto: Anything that mutates a Result and gives you a Result stays. The more colorful ones on Option (take...)
- nmatsakis: This makes sense to me personally.

## str.from_utf8

- acrichto: takes a slice and returns an owned string. And str.from_utf8slice and str.fromutf8_owned. This change removes the borrowed-to-owned bridge. Takes a byte slice and returns a string slice (with the same lifetime - does not copy the data). Also have fromutf8_owned.
- brson: Only one handles non-utf8
- acrichto: Error
- brson: One has a condition that lets you replace a bad character
- acrichto: 
- brson: The non-allocation one cannot handle bad characters. You'd have to allocate.
- acrichto: It always allocates right now, because it returns an owned string.
- nmatsakis: This makes sense and brson's point is good.
- jack: Can't you just return Option of that slice?
- acrhichto: There are versions of that, but the default one returns the value and fails.
- nmatsakis: Do we prefer fallible to returning Result? Maybe this is separate from the question and I should just comment on the PR.
- brson: I worry about this as the default. It would be nice, esp. with utf8, to be able to set something globally to replace unsupported character with a character.
- acrichto: We could truncate by default.
- brson: Then it always succeeds?
- nmatsakis: I'd prefer to return a Result. But maybe I don't call from_utf8 much.
- jack: In python, you have to specify how you want it transformed. Truncate vs. replace with '?', etc. Maybe there should be an alternate version that takes the transform.
- pnkfelix: But doesn't work with slices...
- jack: There's truncate, replace, and fail.
- pnkfelix: Where replace doesn't work on slices.
- acrichto: If we remove from_utf8 with slices on both ends, it always returns an owned string now...
- nmatsakis: Just because it can't do the full range of error recovery doesn't mean we should go with owned strings everywhere.
- acrichto: Then we should probably accept this as-is.
- nmatsakis: Agreed, and we should discuss error handling more.

## dropping libstdc++

- acrichto: I've done it. Impossible on windows, but work on OSX/Linux. LLVM provides exception unwinding, which is all we needed for it. So, if we pair it with libunwind, we can get rid of our libstdc++ dependency. I don't know what else we can do on Windows. So, thoughts?
- brson: Downside is we have to build clang?
- acrichto: If clang < 3.2. So freebsd and OSX, by default, are fine. But we're also including clang, libcxx, and libunwind in the rust repo...
- brson: Long-term important goal to remove libc++, but worried about the churn. And adding clang to the build is also rough.
- jack: Can't you just require clang 3.2 be present on the system? OSX and FreeBSD have it. Debian should ship it.
- acrichto: If I install clang via packages and stuff, it just doesn't work on Ubuntu. The other reason to includ clang is that we get libclang, so we can extract types, etc. from C header files. There's also compiler-rt (instead of libgcc) that we'd get. Lots of other opportunities if we pull in clang.
- brson: If we can't do it at all on Windows, what's our path to getting rid of the C++ mingw dependency on Windows? Is there one?
- acrichto: Not that I'm aware of.
- brson: Maybe building LLVM's libc++ library?
- acrichto: Gives exception unwinding and handling, but not exception resume.
- brson: So, only gcc on mingw can unwind for us on Windows?
- acrichto: libunwind does not seem to work well on Windows. And I don't know of any alternatives.
- pcwalton: Can we use SEH? Or switch to MSVC? We will need to do it to ship a browser on Windows, anyway.
- acrichto: I think LLVM will have issues. But if LLVM can do it, we should probably do that instead.
- nmatsakis: Didn't we have a "no unwind" branch? Could just use return codes instead. It'd be cross-platform, etc.
- pnkfelix: We already have our own calling convention. So is the downside just overhead?
- brson: Could be a stopgap to get away from mingw.
- nmatsakis: We don't know what the overhead is. Something about ack() not optimizing perfectly (TCO missing for our custom convention).
- brson: So, go forward with dropping libstdc++ on Unix & Mac?
- nmatsakis: There are alternative strategies. So, should we take this step, or is that not a way we want to go anyway?
- brson: Drop unwinding on unix, too? Whoooooooo
- nmatsakis: Sure. Why would we have two strategies?
- brson: Slow!
- nmatsakis: Do we know that? Did we measure?
- brson: No :-)
- acrichto: Regardless, we'll have a different Windows strategy.. hrm, out of time.
