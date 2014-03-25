# Agenda 3/25/2014

* RFC: SIMD (cmr): https://github.com/rust-lang/rfcs/pull/15
* attributes in macros (acrichto) https://github.com/mozilla/rust/pull/13067
* counting matches in macros (acrichto) https://github.com/mozilla/rust/pull/12916
* bounds on generic paths (acrichto) https://github.com/mozilla/rust/pull/13079
* fate of collections::List (acrichto) https://github.com/mozilla/rust/pull/13011
* private fields by default (acrichto) https://github.com/rust-lang/rfcs/pull/1
* opt-in builtin traits (nmatsakis) https://github.com/rust-lang/rfcs/pull/19
* if we have time - bounds on type params in structs, types, enums (should we undo https://github.com/mozilla/rust/pull/4660 ? Can someone persuade me #4660 was a good idea please)
* square brackets on macro invocations (huonw) https://github.com/mozilla/rust/pull/12947 (oh whoops, acrichto's added it already...)

# Attending

lars, huon, pcwalton, pnkfelix, niko, azita, acrichto, nrc, brson, cmr

# Status

- brson - 'make dist', buildbot EC2 migration, 0.10 release, docs
- acrichto - sync chan, libsync
- pnkfelix - early/late lifetime bugs, CFG
- nrc: DST - add unsized, remove sized
- nmatsakis: writing some RFCs

# attributes in macros

- brson: Last week, we landed a change to the attribute syntax that should be the last one. Just added a ! between the # and the []. But, that's incompatible with macro_rules w.r.t. the outer syntax. The reason is that ![metaitem matches incorrectly for inner attributes. Idea is to change the definition of attribute rules from attribute matchers to meta matchers, which would just match inside the brackets.
- acrichto: Can re-create the current invocation by putting pattern tokens around the meta.
- nmatsakis: That's clever.
- brson: Would be great if we had meta items anywhere else in the language...
- huon: You can nest meta items inside of other ones...
- pcwalton: Works for me.

# opt-in traits

- nmatsakis: Opened a PR summarizing what we talked about on opt-in traits. No big changes from previous plans. Flapper87 wants to implement it. Are we all in favor?
- cmr: My issue is listed in the RFC - libraries might not annotate the built-in traits that they could be implementing but are not.
- nmatsakis: We could add a lint that tells you which ones you could have implemented but did not... of course, then you'd have to either annotate to silence the lint or turn it off entirely where you don't want them. But that would address the problem. And we'll probably need a lot of lints for "library best practices."
- brson: This is a typeclass problem in general, right?
- pnkfelix: Can't work around it the "usual" way via newtype and doing `impl Send`, right?
- nmatsakis: Only with unsafe code. There's no safe way to say "I assert this is sendable", but that's by design.
- brson: How does Share work with this, nmatsakis? 
- acrichto: Discussed on IRC today. Unsafe<T> is Share. 
- nmatsakis: That's also in the RFC. Unsafe implements ALL traits...
- acrichto: Except Copy?
- nmatsakis: Why not? Minor point, but we should address it.
- acrichto: We don't have any types today except for Mutexes with Share where we want to be able to opt-in... but maybe Send and Copy should be as normal for Unsafe.
- nmatsakis: I was debating about Send.
- pcwalton: Really want Send to be opt-in.
- nmatsakis: Just talking specifically about what the Unsafe type should do, I think. Not clear what the right way to take something non-Send'able and make it Send'able... For now, I'll just say Share.
- brson: Let's do it. Acrichto will merge this into the RFC and we'll move it forward. 

# SIMD

- cmr: Idea is to add a hack type/macro/thing called SIMD. That's the most controversial part. It extends our SIMD support from the simple attribute to supporting field projection and shuffles in a way that integrates more nicely with LLVM. There's some code that implements this and it looks good - just needs test and docs but is otherwise ready to go if we think it's the way forward.
- acrichto: Definitely behind a feature gate, right?
- cmr: Yes. I think SIMD itself is behind a feature gate today, too.
- nrc: Does it work by allowing macros in types? Or more hacky than that?
- cmr: It's got a special case thing since we can't use types in macro positions...
- nmatsakis: So SIMD is a keyword?
- nrc: I like the syntax for SIMD, but it would be nice if we could do this via the macro system.
- nmatsakis: I'm wondering why we would want this syntax, because if we could make it a macro, presumably it expands to something and we could just be typing that right now.
- nrc: I think the idea is that it'd expand to a whole bunch of intrinsics and low-level stuff...
- brson: That should be spec'd here.
- huon: asm expands to stuff you can't type except in macros. The current implementation is not intended to be the final one.
- cmr: Has some typing issues because some things are required to be constant or LLVM can't handle it. This is also not a normally-behaved type, and so these rules are intended to make that evident.
- nrc: Is that spec'd in the RFC?
- cmr: Yes.
- pnkfelix: Meta question: Do we have a way of marking RFCs as for-1.0 vs. feature-gated, etc.?
- nrc: No, but it should be stated explicitly in the RFC.
- nmatsakis: Lots of existing SIMD approaches. Still not sure which fits best for Rust... this RFC is one, but I don't know yet.
- pnkfelix: Future features (e.g., associated items) that would invalidate any decision we make here?
- nmatsakis: Not associated types. But there are some C++ libraries to do SIMD-level stuff that might be worth looking at, though they likely use features we don't have. Also, CilkPlus has various operations, as do other Intel compilers, for vector ops. And of course there's the JS work, which is semi-related. I don't know that any of that is any better than what's being done here.
- brson: Well, details aren't clear to me and some of this looks scary, but it would be good to have people working on SIMD, so long as it's under a gate. I'm just a little hesitant about this proposal. I'm also not sure how this shuffle syntax is implemented.
- cmr: So far, it looks at the field and then checks if it's a recognized shuffle for that type (using the field name) and then emits the proper intrinsics. These field names correspond to positional elements - the syntax is straight out of OpenCL.
- brson: Init syntax looks like a vector, but it's a struct?
- nmatsakis: They're both.
- nrc: It's a struct-tuple-vector-thing.
- acrichto: Would be nice if it was more of a library than language thing...
- nmatsakis: I want SIMD syntax. But I don't know that this is the one we'd choose if we have to special-case the language.
- brson: I don't like that SIMD required integration with lots of different parts of the compiler.
- cmr: Carter had lots of feedback in IRC. He said that doing it in a library often can't be as nice, particularly when SIMD vector sizes are extended and you have a blowup of the swizzle masks, etc.
- brson: But we do have macros! Though those are not as nice as built-in syntax.
- pcwalton: I know people say you can't do it with macros and as a library, but I'd like to see the attempt.
- cmr: I'm just worried about having 4096 field-like things.
- brson: The nice thing about doing it in macros is that it's encapsulated...
- pcwalton: I really prefer macros here. I've worked with SIMD in shaders a lot. It's fine if it's not beautiful code.
- nmatsakis: I want to see a good summary of alternatives. I'd like to know: if we did it in macros, is there an idea how that might work? Maybe see an algorithm written in different ways? This just seems like a really big feature and I don't know what's right. I know feature gates keep it separated, but...
- pnkfelix: I'd also like to see it as functions instead of  attributes...
- nmataskis: I also don't get the 3-element SIMD vectors that show up in this...
- cmr: I think that's how LLVM lowers it - fills in an empty element for the fourth one.
- brson: We do not have consensus to land this, but we all want SIMD. What specific feedback can we give aatch?
- nmatsakis: What's the associated PR?
- cmr: There isn't one, but he has an associated branch in his repo.
https://github.com/Aatch/rust/tree/simd-support
- brson: So, see nmatsakis' requested summary plus possibly an idea of what it looks like as a macro.
- nrc: Just the specification of the alternatives is fine, not necessarily asking him to re-implement it three times.
- nmatsakis: Keep in mind that macro rules can't count.
- brson: Recursively, can't they?
- nmatsakis: But you can't have the number 4 and then... meh, this is a red herring.
- brson: Specification + implementation that uses macros for shuffles/swizzles. cmr, can you follow up with aatch?
- cmr: yes!

# [] on macros

- huon: Recently added {} on macro invocations. There's another PR adding [] as well, so you can have vec![1,2,3]. 
- brson: So this supports the last RFC?
- acrichto: Independent.
- pnkfelix: Use case is that our current vec macro uses parens, but we really want square brackets.
- brson: Makes sense to me.
- acrichto: Uninvasive and sounds good.
- nmatsakis: It's fine.
- pnkfelix: Should this be an RFC?
- nmatsakis: In principle, but it's so small... 
- pnkfelix: Just make it clear we're making an exception from the process.
- brson: Any forward-compat problems?
- acrichto: Only if [] mean different things in the future...
- pcwalton: I'm fine here.
- brson: Let's do it. Huon, you can take care of it.

# counting matches in a macro

- acrichto: If you have a * or a + (big use case is in the vec macro) you can count how many you matched and get a number literal out of it. Only supports counting one variable. And the syntax instead of a $ is a #.
- brson: Any major use case this is blocking?
- acrichto: No. I've written many where I wanted to count things but have always been able to do so.
- nmatsakis: There is a bug he's trying to address...
- brson: More effective vec.
- pnkfelix: There's a workaround huon wrote...
- brson: vec could also be a syntax extension. Assuming it should even exist.
- nmatsakis: I'd like to get the opinion of jclements, but I'm not sure why this would cause a problem.
- cmr: I CC'd pauls
- nmatsakis: I think this needs an RFC.
- brson: So, we all like this feature, but we need to see an RFC. acricho, can you follow up?
- acrichto: Yes.

# bounds on type parameters in structs/types/enums

- nrc: This came up while implementing the unsized keyword for DST. That we do have to check because if we have a type parameter X and a field in a struct of X, we have to know if it's sized or unsized. It seems odd as to why our current approach is a good one:
email from nrc:
https://mail.mozilla.org/pipermail/rust-dev/2014-March/009175.html
- nmatsakis: I think we should allow bounds, and I have a reason.
- nrc: Does anyone not?
- acrichto: What does it mean to have bounds?
- nrc: Structs with bounds on the type parameters mean that we check anywhere we use that struct when it's used e.g. in an impl that the bounds are satisfied and when you use it, that the bounds are satisifed in the impl.
- nmatsakis: Could also do it when you make instances of the struct. That alone is sufficient to guarantee that you always satisfy those bounds.
- nrc: We already check per-field...
- cmr: Only on method calls in impls.
- nmataskis: Exactly. The reason I thought this is a good idea besides unsized is that when we define destructors, you have this problem wherein the destructor would like to know about the bounds on the type, at least if they apply uniformly and not just to ones where the types meet certain bounds... If the destructor only applies to T where T:Send, what does that mean for structs where T is not Send? Another is to say that there just is no destructor if T is not Send. But that makes me nervous...
- acrichto: pcwalton, you mentions haskell used to have this but is now strongly against it?
- pcwalton: Somebody posted on the mailining list about it, but I havent' gone deep yet.
- nrc: I think the example was if the bounds are more precise. So if you have a HashSet, you only need T to be Hashable for get/insert, but not for size. But that doesn't matter for Rust, so I'm not sure whether the Haskell argument isn't relevant to Rust.
- nmatsakis: Yes, but minimal set of bounds on your types means that some methods now have stricter bounds. Seems fine. I think it's good to have bounds on your struct.
- pnkfelix: We can also add a lint for the Haskell case, if we decide we care later...
- nrc: I just don't like this because it means we have two kinds of type variables: type variables on traits are one kind and type variables on structs are another and are checked completely differently. They have different behavior, arrows happen in different places... it's just a whole bunch of extra hidden complexity. Type aliases, structs, and enums. Type aliases do get complicated, because if the variables are not 1:1 with what you're aliasing to...
- cmr: Wouldn't it be checked after the alias is resolved?
- nrc: I guess you can always infer the bounds. Kind of a bikeshed.
- nmatsakis: Type aliases just go away - the typechecker doesn't even know they exist. Anyway, I'm in favor of this.
- cmr: I've traditionally not been in favor of this on IRC, but it does seem like it would be more convenient than typing ":Send" everywhere...
- acrichto: It's more verbose now - have to put it in more places.
- nmatsakis: If we're going to allow the unsized keyword, that's a bound. It's not optional, unless we want to special-case that...
- acrichto: I'm in favor of this because DST requires it and it seems weird to only have it in for the unsized one.
- brson: I have a question for pnkfelix based on reddit post for another keyword for unsized. It was like "this is the default set of bounds for T". 
- pnkfelix: nmatsakis and I have been talking about that, but I'm not sure what that means - I don't think we can really add new bounds post-1.0.
- nmatsakis: I do like that proposal because it's good for handline uints, higher-kinded types, etc. Good place to write down a kind. I originally agreed with you, brson, but you'd have to make every type parameter only have the defaults to avoid having new automatic bounds appear post-1.0.
- brson: Didn't mean to derail, but that helps.
pnkfelix: FYI: we were above discussing this post: http://blog.pnkfx.org/blog/2014/03/13/an-insight-regarding-dst-grammar-for-rust/ (or this reddit discussion, I guess: http://www.reddit.com/r/rust/comments/20bfc1/dst_syntax_proposal_to_avert_a_bikeshed_use_type/ )
- brson: nrc, do you have the info you need for this?
- nrc: Sounds like nobody really objects, so I'll write up an RFC and see if there are any wider objects. Doesn't block DST for now, so I can go forward.
- brson: Sounds good.

# private fields by default

- acrichto: Getting close to merging. One major alternative linked on the RFC. Idea was instead of "private everywhere", it's like class but calling class abstract instead. In his, field-level visibility cannot be controlled fine-grained. Either all-public or all-private. Idea of an abstract struct or enum, where everything is private to a module. Everything abstract is private everything not is public. Few keywords to make it more terse. In my opinion, it sounds like something we could add on later, but I still feel private fields are the way to go. The argument was the we don't need field-level visibility, but I think we need it for the Formatter structure. Any opinions from others?
- pcwalton: I agree with you.
- nrc: I do as well.
- nmatsakis: I think in the alternative proposal, it's the wrong defaults, since people have to type more to get what they usually want. I'm also worried about adding more modifiers. So, basically, I agree.
- brson: Are we going to reserve priv? And, does this rule out private enum fields? So, in a newtype struct, are we ruling that out? Newtype struct: their fields are public....
- acricho: No, tuple structs now have private fields.
- brson: Good!
- acrichto: This is a very invasive RFC because it flips defaults everywhere.
- brson: Nice. Somebody should merge this RFC and talk with Gabor. It'll be me.

# collections::list

- acrichto: Currently GC singly-linked list. Why? What should the API be? PR to make it an owned singly-linked list, but this is similar to a vector. I'm kind of thinking we should just throw this type away. What are feelings others have?
- cmr: I'd be fine, because we already have DList, which is our owned linked list.
- nmatsakis: I agree. Should have a persistent library collection... later.
- nrc: We have a linked list. That's plenty.
- nmatsakis: I'd be willing to get rid of DList, too...
- acricho: Just the List type for now.
- nrc: I think we should have one in the library.
- pnkfelix: So libarena would have a private copy of List?
- acrichto: Or we'd turn it into vector.

# bounds on trait paths

- acrichto: If I want Send as a bound on the following trait, it parses badly today, my PR changes to move the Send to the end:

```
trait Foo<A> {}
// today
fn foo(arg: ~Foo:Send<A>)
fn foo(arg: ~Foo:(Send<A>))
// tomorrow maybe
fn foo(arg: ~Foo<A>:Send)
fn foo(arg: (~(Foo<A>):Send) )
// valid type (in a DST world)
type T = extern fn foo(Foo:Send);
```

- acrichto: Bounds will always be at the very end, regardless of generics or not. How do people feel? Make an RFC? Merge it?
- nmatsakis: I'm in favor, but there's some ambiguity with :, right? To do with optional argument names?
- acrichto: Still in place, and not addressed by this in any way. Could have a type that looks like the extern `foo` above and have no idea what's going on. But I think that issue is different.
- nmatsakis: Maybe we should just get rid of optional named arguments. They're really annoying in many places.
- cmr: I often forget they're even optional...
- acrichto: They often surprise me.
- pnkfelix: Can you now write the whole type in parens, at this point? Is that legal?
- nmatsakis: That is not legal (the part B of "tomorrow maybe"). It's just unfortunate for cases like C where if you can't remember the precedence rules you can always just explicitly group things by adding parens.
- brson: Yes, just worried about where you can actually stick parens. This fix seems pretty uncontroversial, right niko?
- nmatsakis: Yes, I like it.
- brson: Let's do it.
