# Agenda 12/10/2013

- "enum mod" (acrichto) https://github.com/mozilla/rust/issues/10090
- use {foo, bar} (acrichto) https://github.com/mozilla/rust/pull/10808
- libcxxabi/libunwind to drop libstdc++ (acrichto)
- unofficial nightlies (brson) https://github.com/graydon/rust-www/pull/17 https://github.com/mozilla/rust/pull/10805
- Using types to inform inference https://github.com/mozilla/rust/issues/10834
- expansion of struct patterns (brson) https://github.com/mozilla/rust/pull/10833
- formal grammar for 1.0? (pnkfelix)
- RevOrd (pcwalton) https://github.com/mozilla/rust/pull/10860
- trailing commas (pcwalton)
- 'self? (pcwalton)

# Status
NB: "tvland" refers to remote people in the meeting, since they appear on a TV screen.

- acrichto: landed LTO, fixing races in std::comm, extracting libgreen/libnative
- pnkfelix: borrowck bugs, GC
- brson: android automation, tech writer recruiting, workweek and other planning
- nmatsakis: lifetime fixes #5121, lifetime of temporaries #3511
- pcwalton: new cell, Pod trait, "box", changing @mut to use cell, servo stuff.

## enum mod

- acrichto: Several people have asked for this. Enum variants namespaced in the same module. But enum mod would create a new module with the variants scoped under it. People want this so the variants are scoped in the name of the enum. Some want both. Some want one or the other. Those seem to be the sides. It looks pretty clean to me, but people who want one or the other have a good point.    
- nmatsakis: Not a new module. New namespace.
- dherman: enum mod wouldn't declare a module? Weird.
- nmatsakis: Not because enum mod foo => a type, not a module.
- dherman: Am I the only one who thinks this is crazy?
- pnkfelix: Can you repeat the example?
- nmatsakis: If you write:
```
 enum mod Foo 
```
Then Foo is bound to the enum type you created, not a fresh module named Foo. Today, a name cannot be bound to both a module and a type.
- pnkfelix: So Foo would still be how you specify the namespace?
- nmatsakis: Yes, `Foo::variant_1`, etc. The definition does not desugar into a definition form we have today. I sympathize, but this feels like it is not a high priority for me. Maybe a better keyword choice would help?
- brson: Seems like scope creep.
- acrichto: The community sounded like if we say yes but if we don't prioritize it, somebody will land it for us. So, if we approve the change, somebody will do it.
- dherman: It could be "to be decided." Which is not "yes."
- nmatsakis: The `mod` keyword feels like a C-like abuse of keywords ala static.
- pnkfelix: My model has been that types that create namespaces desugar in my head into a module form anyway (for structs). 
- nmatsakis: There is a sort of precedent for methods. I'm OK with it being a new thing; just wanted to clarify that it is not, in fact, a module.
- acrichto: So `enum mod` not accepted as the syntax in the issue would be OK, if it allows the community to bikeshed it more?
- jack: What do we lose by making the variants always scoped to the typename?
- acrichto: It would make a lot of people unhappy in the transition
- jack: Because you'd have to change your use?
- acrichto: People would use expr blah
- jack: Have to change your use to import the variants directly.
- acrichto: That goes against the style we currently have. Normally just have a capital-thing. This would bring in things with two capitals (e.g., "Option::Some")
- pnkfelix: Inner module and expose the variants from that?
- acrichto: Then the documentation looks really weird. But that's a valid workaround.
- pnkfelix: I don't want to make language decisions based on current doc tool limitations.
- nmatsakis: I don't know if it's just a tool limitation or actually the design of the language. This feels like change for change's sake.
- brson: Too late for this change. Quota is "one catastrophic change per quarter."
- acrichto: Nobody is in favor of this; many seem against. Won't fix the bug?
- pnkfelix: Yes. Mostly against out of conservativeness.
- brson: Maybe revisit later...

## nightlies

- brson: PRs to add pointers in our docs to unofficial community binary nightlies. Do we want to be recommending community binaries? Also, our plan is to add our own nightlies at some point, but who knows when we will get to that. Does anybody have opinions?
- acrichto: Binaries are special. Arbitrary code running on your computer...
- nmatsakis: Not that uncommon to point people at unofficial mirrors, etc. 
- acrichto: Mirrors have signatures and stuff so you can verify the binaries.
- brson: For Rust, it's a best practice to use master. We could get into a position where we are no longer distributing what people are using. We'd like to be the ones distributing what people are using.
- brson: I'm just going to say no on this.

## using types to inform inference

- nmatsakis: We have evolved a few different schemes that cope with the same issues. Where the compiler attempts to infer what is legal, we sometimes want additional restrictions (usually unsafe). NoFreeze, NonSend, etc. that tell the compiler about Cell or RcMut. Also, the special non-copyable type that can be imported. Finally, inferring variants for lifetimes (e.g., where they're unused but you should act as if they are because the pointer is unsafe). I think we should do these through the same means. Proposing: dummy types that are special with zero-size. Other options are annotations or special syntax. Right now, we use or have proposed all three. I like the types because you can attach comments and documentation to them. We can easily extend them or add more of them. It's less intimidating than special syntax. I'm just not a fan of annotations because of their ad-hoc feel. They're not discoverable, don't appear in rustdoc, ect.
- brson: Do these behaviors propagate through normal type inference rules? Just by putting this in a type, it all works?
- nmatsakis: Yes. Either the kind inference does it or the lifetime inference does. The constraints just propagate out.
- pnkfelix: Because I'm not clear on this, it doesn't seem consistent. Noncopyable propagates out into containers, but covariant lifetime affects the type parameter...
- nmatsakis: Yes, the variants of the type paramter with respect to the containing type.
- acrichto: I like it, but construction of these types gets annoying since you need a unit constructor. With attributes you don't need to do it. Then again, all of these types are library types and you do not use a direct constructor.
- nmatsakis: And having a dummy name for the field, too.
- brson: Static constructors would be available for them, right, so you wouldn't have to use new?
- nmatsakis: That's a good case for noncopyable, etc. But for contravariant lifetime I'm not sure. I imagine we'll end up with them for type parameters.
- brson: Can covariant lifetime not be expressed as an attribute?
- nmatsakis: Not sure how you'd name the lifetime. Maybe with a string? Feels kinda nasty. With this, we just need a small hack to the inference engine, but beyond that everything falls out. Instead of `#covariant_lifetime"'a"`.
- brson: Not opposed to this, though I thought we'd go the opposite direction.
- nmatsakis: The main change is w.r.t. the NoFreeze annotation to types.
- brson: I don't mind.
- pnkfelix: I'm OK with this.
- brson: Done.

## formal grammar?

- pnkfelix: Follow-up to the e-mail. Do we need the grammar for 1.0, or can it be post-1.0? Do we feel comfortable asserting that whatever the current parser happens to encode is the language we support for the future? Patrick: when you and john were looking at it earlier, I know he had to tweak to make it LL(2). Were there other changes?
- pcwalton: It's LL(1). It's important to clarify. If you left-factor enough, it's LL(1).
- pnkfelix: Do we belive that based on the parser? We have no left recursion...
- nmatsakis: What about self parameter and arguments that requires arbitrary lookahead?
- pcwalton: I left factored it enough.
- nmatsakis: Because they can only be 1 deep?
- pcwalton: Yes.
- pnkfelix: Anyway, what do people think?
- pcwalton: Formalization of the grammar is no more of a priority than any other part of the language. We take our chances.
- dherman: It's a risk assessment. It's not something that's clear - need a judgement call.
- pcwalton: If the parser is accepting something it shouldn't, so what?
- dherman: My analysis is that the risks are the parser doesn't accept something it should => no problem. If it does accept something it shouldn't, now you have to decide if users relied on it. If not, just remove it safely. If they are, you then have to decide that you want to break code and upset customers. That's the worst case scenario, either way. So, how big is the risk? Pcwalton is saying that the risk is pretty low, and we should prioritize other 1.0 work. I'm fine either way.
- pcwalton: We already did some work formalizing the grammar, though not lots of test cases except to check that it accepts all of the Rust code that we've written. The language has changed slightly, but we've done enough work for me to feel comfortable with the risk.
- pnkfelix: Agreed. Since you did that work, we're fine.
- brson: That grammar accepted the entire suite? We should commit that...
- pnkfelix: Different ASTs... but that's generally true.
- brson: Grammar, not a 1.0 priority.

## imports

- acrichto: If you have foo and bar at the top of a crate, you can't say `use {foo, bar}`. Do we want to accept this? You can say `use foo,bar` but nobody does that. You can say as many viewpaths as you want, separated by commas.
- brson: Shouldn't we remove that?
- nmatsakis: Seems inconsistent with lets to me.
- acrichto: You can import multiple items at the end of the path `use path::{stuff1,stuff2}`...
- nmatsakis: I mean that the `use {foo, bar}` seems inconsistent. Fine with colons. The current setup where paths in bodies are not absolute causes confusion.
- pnkfelix: I've been confused by it.
- nmatsakis: Maybe they should just be absolute. Then there would not be two kinds of paths.
- acrichto: Accept use statements without leading :s?
- pnkfelix: For relative only.
- acrichto: Didn't we have relative and remove them?
- nmatsakis: Have them. Self or super form or something.
- pcwalton: Yes, self and super. Sometimes wish we didn't have them.
- acrichto: Remove them?
- nmatsakis: Should formalize the name resolution algorithm...
- pcwalton: Yes. It's decidable without self/super. But I'm not sure that namespace lookup with self, super, and `pub use` is decidable. Definitely with globs it's not.
- dherman: Yes, much higher risk. We've bashed our heads against multiple times. And the type system is far more sensitive to mistakes in name resolution semantics than the grammar. 
- pcwalton: Yes, and the algorithm for name resolution is bogus because it was trying to solve globs, `pub use`, and the rest.
- dherman: Multiple researchers looking to collaborate. Maybe get people working on name resolution separate from type system? Maybe not proving things, just writing a precise specification of the algorithm would be great.
- nmatsakis: True! But I think the people who have expressed interest so far are into this. Maybe Matthias's students. But his are into language design...
- larsberg: Amal's student. She's a typetheory person (semantics of state, etc.)
- pcwalton: Self & super
- acrichto: We use this stuff a lot.
- dherman: Your point on Sam is good. Lots of research on recursive modules, but not on globs. There should be a publication there... though what's the theorem there? Maybe just decidability is interesting. We can talk to Amal, Matthias, Sam about it.
- pnkfelix: Is the claim that this should not work:
```
mod foo {
        use m::id;
    mod m {
        pub fn id(x:int) -> int { x }
    }
}
```
because it does for me....  (or maybe not, I was putting contents of "foo" into a file foo.rs)
- pcwalton: That does not work. If it does that is a bug.
- brson: Decision here?
- pcwalton: Should we feature-gate self/super?
- pnkfelix: Not opposed.
- brson: I like to get rid of things.
- acrichto: Add this and remove the comma syntax.
- nmatsakis: +1
- brson: Agreed.
- pcwalton: Works for me. I just want to make sure name resolution is decidable. For most of our history it was not.

## adding new kinds of struct patterns

- brson: Before merging, do we want this (#10833). Makes ref struct patterns easier.
- acrichto: I've always wanted this; it's awesome.
- pcwalton: Agreed.
- brson: Comments from "tvland"?
- pnkfelix: Sure.
- nmatsakis: Affect LL(1)-ness? I don't think so. So OK.

## trailing commas

- pcwalton: We don't allow them sometimes, like in struct patterns. Can in literals but not patterns. There's no good reason for this. Also, should we just accept it in tuples and argument lists as wel?
- nmatsakis: Everywhere or nowhere. And I like them. Everywhere.
- jack: Makes code generation easy. +1
- pcwalton: Good.

## RevOrd
- pcwalton: Wraps and reverses the order. I'm in favor. Other opinions? Don't feel strongly about it, though.
- acrichto: Worried about feature creep.
- brson: Me too.
- pcwalton: And, "tvland"? Too slow, I'm closing it.

## revisiting unwind

- acrichto: Ran out of time last week. I'm still in favor of doing this. The most contentious parts are our Windows behavior and using an older clang on linux distributions.
- brson: So this would involve using clang to build all of our C++ code? Or just libcxxabi?
- acrichto: Only need for libcxxabi
- brson: Might make makefiles easier if we just use one compiler.
- acrichto: The makefile additions are small. But we do have to manage two compilers. The worst thing is having to build clang everywhere.
- brson: And out of tree, too.
- acrichto: If we find it out of tree, we'll use it. If not, we build it in-tree.
- brson: That seems like a burden. I want control of everything.
- acrichto: One solution would be binary snapshots of only LLVM and clang. Then, when we download a rust snapshot we also download LLVM and clang ones. That would be kind of nice except that we have to do it for all architectures.
- brson: I just don't really want to tackle this problem right now. Very fiddly, more complication.
- pcwalton: I'm sympathetic to that argument. I have bad memories of the dark times when we had to build clang.
- brson: And doesn't help us with building the toolchain on Windows. Just cleanup for a toolchain that's already pretty stable. It's an important problem and we should get there eventually, but is it important now?
- acrichto: Main use case is for embedding Rust. We're at a much better place than we were 6 months ago.
- pcwalton: Can't we just turn off unwinding?
- acrichto: Have to recompile everything. Only need it for rust-cxxglue.
- pcwalton: Maybe weak bind against those and turn throw into an abort?
- acrichto: Still have to distribute a separate copy of libstd...
- pcwalton: Couldn't we let the author choose? Let's take this offline.
- brson: What will embedded people do if we don't tackle this?
- acrichto: people don't want a libc++ dependency, in general. If we can get a nice compile-time option to change to an abort failure, that would be a great stopgap.
- pcwalton: That would be a 30-40% improvement in rustc build time.
- acrichto: Would still generate landing pads.
- pcwalton: Why?
- acrichto: Maybe upstream would have them. Yours wouldn't...
- pcwalton: As long as librustc doesn't have them, it's a huge perf improvement.
- acrichto: Could have a separate static library we distribute... I'll pursue this option.
- pcwalton: That was what we were thinking about a year ago on this problem :-) 
- brson: abort is not the same semantics as fail, though.
- acrichto: We should never do failure by default. Unwinding is useful. But some places where we don't want to do it by default.
- brson: There are tons of places in the standard library where we want to fail recoverably.
- acrichto: Shouldn't be architecting for failure and recovery.
- pcwalton: If that's not the case, we need catchable exceptions. I want it to abort the whole process. The compiler does not need to be recoverable.
- acrichto: Option out of unwinding is really unfortunate. It's provided by libstdc++, which is unfortunate.
- pcwalton: Also codegen and bloat issues.
- acrichto: There are cases where we want unwinding.
- pcwalton: We need to support several cases.
- dherman: This feels like a really important thing to be clear and crisp on about the philosophy, design, and intended programming model for Rust 1.0. It answers questions about what is a task? What does failure mean? Ties into 1:1 vs. M:N scheduling, ties into where we build and run. We need to be very careful to be clear about this. Need to get consensus and be able to communicate it well.
- pcwalton: Failure should be recoverable at the task boundary.
- acrichto: Shouldn't promote dropping unwinding completely.
- dherman: Does recoverable failure mean that we have to fork all libraries, then? With two separate builds?
- pcwalton: If we intend to allow people to depend on unwinding, the language should have catch.
- dherman: So fail should be abort?
- pcwalton: Yes, but at a coarse level, you should be able to use task::try. But failure should not be normal.
- dherman: Just one failure construct and deciding at the catch boundary if it's abort? Does it mean we just see if we're in a recoverable context or not?
- pcwalton: Not a performance issue - fail is super-expensive anyway.
- nmatsakis: Get the perf benefit of not generating unwind code.
- dherman: If you want the guarantee of no unwinding, though, have to not link against any code that uses task::try, right? It's just that all of this stuff is not clear to me when I put on my "Rust User Hat." How do I understand what I can and can't use when I'm writing either libraries or kernel code, what can/can't I link against, etc. These descriptions make it sound like things are forking.
- acrichto: Then we should abandon this. There should be a single version of a library. You might decide not to use a library because of your platform, but there should be nothing in the language that forks them.
- dherman: Can you make it at least clear to developers which features are safe in which contexts? e.g. "kernel-safe" vs. not.
- nmatsakis: Let's table until next week!
