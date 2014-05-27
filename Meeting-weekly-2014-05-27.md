# Agenda 5/27/2014

* doc generation in default make target (pnkfelix) https://github.com/mozilla/rust/issues/14424
* Should Process::drop wait for the child? (huon, although alex knows much more) https://github.com/mozilla/rust/issues/13854
* lexical syntax changes (cmr) https://github.com/rust-lang/rfcs/pull/90
* ufcs (nrc) https://github.com/rust-lang/rfcs/pull/4
* Remove markers and replace with a default rule for unused type/lifetime parameters (nrc/nmatsakis) https://github.com/rust-lang/rfcs/pull/12
* RFC for attributes on statements and blocks (not match arms) (nrc) https://github.com/rust-lang/rfcs/pull/16
* Space-optimize `Option<T>` for integral enum `T` (nrc)  https://github.com/rust-lang/rfcs/pull/84
* Macro expansion in patterns https://github.com/rust-lang/rfcs/pull/85
* Plugins and loadable lints https://github.com/rust-lang/rfcs/pull/86 https://github.com/rust-lang/rfcs/pull/89
* Unboxed closures (pcwalton) https://github.com/rust-lang/rfcs/pull/77

# Attending

aturon, acrichto, kmc, pcwalton, brson, luqman, bjz, nrc, huon, erickt, pnkfelix, cmr, zwarich, nmatsakis, ecr, dherman, azita

# Status

- aturon: wrapping up first cut of library doc; path API; stabilization attributes; rustdoc UX; RFC writing
- brson: VPC porting
- nrc: DST
- pnkfelix: borrowck atop cfg; Allocator RFC drafting/revising
- acrichto: librand, libcollections, rustodc inlining
- cmr: repr-on-structs, grammar
- erickt: libserialize rewrite

# Action Items

- cmr: update lexer rfc for not dropping whitespace
- niko: New UFCS RFC
- brson: Merge https://github.com/rust-lang/rfcs/pull/85
- pcwalton: Update https://github.com/rust-lang/rfcs/pull/77 to reflect closure plan

#  make install builds some docs

- pnkfelix: Basic issue is: if you run make, make install, then make install does not complete immediately.
- pnkfelix: it takes a while to run due to doc generation
- pnkfelix: My suggestion is to change the make target, but people who don't care about the docs might object to make taking longer
- luqman: It even does it when you pass disable-docs. That's a bug.
- brson: So adding this will add a minute to the build time. I think we should just do it.
- brson: Or just not gen docs for rustc.
- acrichto: make install doesn't need these docs install. Shouldn't be a dependency!
- brson: make all depends on docs; make install depends on all
- acrichto: we should definitely remove it from make install
- pnkfelix: I was assuming we want it for make install
- brson: docs should be installed
- pnkfelix: I'm not sure what the action item here.
- pnkfelix: I'm trying to decipher the last fragment of conversation. The message is inconsistent.
- pnkflexi: acrichto is saying make install shouldn't build/install docs.
- pnkfelix: Are we building the docs but not installing them?
- brson: I think it's subtle. Long-term, we should expect that docs are installed; that's very typical.
- brson: Short-term, we can break the dependency and not build docs on install.
- brson: Nobody has made a firm commitment here.
- pnkfelix: The other piece is: luqman's pointing out that --disable-docs is not effective.
- luqman: seems easy to fix.
- pnkfelix: Would anyone object if make does build the docs (for rustc)?
- acrichto: It can do it in parallel, so it won't be that bad.
- pnkfelix: OK, no one objects, so I'll add it as a background task.

# Process::drop

- acrichto: We said we shouldn't be blocking in destructors. But this is dropping in the dstructor.
- acrichto: But if we do not wait, we will assuredly leak, which is also bad.
- brson: We decided previously that we don't want to fail in destructors. This is slightly different.
- brson: With the pattern where destructors are inclined to fail, we want to provide a standalone method that would do what the destructor would normally do, and then defuse the destructor bomb. Can we do that here?
- acrichto: the problem is that you'll leak if you don't wait.
- brson: Maybe the docs should say: you should call "wait" before the destructor.
- brson: Generally, you should not block in the destructor (just like we prefer not to fail). But this could be an exception.
- brson: But we should have docs saying to call wait.
- acrichto: But then how to you make a daemon (i.e. a process that outlives yourself)?
- nmatsakis: You could call detach, or set it up as a builder.
- brson: We could spawn a thread for reaping processes.
- nmatsakis: Isn't that sort of orthogonal? If we spawned a thread to reap processes, what about reaping the daemon? Isn't the point that you *don't* want to reap the daemon?
- acrhicto: I was thinking we would not call wait in the destructor, with a stern warning about leaking.
- cmr: I wrote in the issue that you could use `must_use`
- acrichto: That won't work. If you call any method, that's considered using.
- nmatsakis: That also doesn't handle the failure case. If you fail before you would've waited for it, it will leak. Is that not a concern?
- nmatsakis: How would you possibly be secure against leaking except by calling wait in the destructor?
- acrichto: Every time you get a SIGCHILD we reap it.
- nmatsakis: It seems like we should call wait in the destructor, unless you set it up as detached.
- acrichto: After it's been spawned, we could consume and detach at the same time.
- brson: would that require active reaping on our side?
- acrichto: You have to explicitly opt in to it.
- acrichto: As long as you are alive, you are leaking all your daemons; they are consuming resources when they exit.
- acrichto: What normally happens is the kernel reaps it, but you have to exit for that to happen.
- acrichto: You should set it up so that you exit promptly.
- brson: Why not just push work over to a worker thread, which we already have for IO?
- acrichto: not sure how this would work for libgreen
- pcwalton: libgreen already has a worker pool
- nmatsakis: Usually you have to do this because there's some data that comes out of the process, like an exit code. That's all we'd have to preserve.
- brson: What actually leaks if you don't wait?
- acrichto: The kernel probably has some resources, like the exit code
- pcwalton: The pid.
- acrichto: If you explicitly opt in to detaching, you are explicitly opting in to leaking these resources.
- brson: It'd be a safe function that says "leak".
- nmatsakis: I would prefer to block, based on what I heard.
- brson: Let's block, and leave the option to add a detach method in the future.

# Lexer changes

- cmr: The biggest change is that rather than doing any work in the lexer that we do today, the outcome of the lexer conceptually is just spans and the token the span represents.
- cmr: that includes comments (nice for pretty printers) and excludes things like escaping characters in strings and parsing numerics.
- cmr: it doesn't change our lexical syntax itself, but it changes how it works (and the input to the parser)
- brson: What's the motivation?
- cmr: Testing rustc's lexer is very hard; it does a lot of work.
- cmr: Also, given a token stream, you cannot recreate the input, because escaped characters in strings are clobbered.
- brson: So how will the  new lexer cope with escapes and these other things handled manually?
- cmr: It won't.
- brson: Something has to.
- cmr: The parser.
- acrichto: The output looks like "this span corresponds to this token"
- brson: We're going to still lower these escapes at some point, but it will be at the AST level
- brson: cmr, you're proposing to do this work?
- cmr: Yes.
- acrichto: Is there prior art here?
- brson: the comments issue worries me -- in the AST?
- cmr: they wouldn't go in the AST
- nmatsakis: I presume there's a filter prior to the parser that would drop that
- pcwalton: this would help for rustformat
- dherman: There's a million other tools people would be able to build on this
- nmatsakis: This seems like an improvement. As long as we don't try to filter the stream in the parser -- that way lies madness.
- cmr: I don't want to do that in the parser.
- nrc: Is whitespace the only thing that's dropped?
- cmr: yes
- nrc: But given a span, you can infer the whitespace?
- cmr: yes
- nmatsakis: is it clear that we want to drop the whitespace? I know we can infer it.
- acrichto: [something] wants to keep whitespace
- nmatsakis: it doesn't make any difference to the parser.
- nmatsakis: but it would be nice for the lexer to be completely faithful to the document, then you drop what you don't care about.
- nmatsakis: if you want to know "was this on the same line" it's easier to tell
- nrc: May also need to know the *kind* of whitespace
- cmr: oh, I forgot about carriage returns. What do we want to do about them?
- cmr: Eridius had a PR about this -- handling CRLF better in string literals/comments
- cmr: today, we do bad things on windows
- cmr: I'm inclined to say don't use carriage returns in rust code
- nrc: We should handle them. If you wan to be usable on windows, you have to handle them
- huon: what about macros? do they get a token tree rather than an AST?
- huon: we'll probably want helper to keep macro writing sane
- cmr: I think we can just skip all the junk when making the token tree
- nmatsakis: We derive the token tree from the token stream
- cmr: I thought so as well.
- kmc: Uncomment macro -- that would be a disaster
- huon: keeping whitespace would let you have a Python macro
- <hesitant giggles>
- cmr: token trees are created out of the token stream. It'd be easy to strip them out.
- huon: not converting strings to ints and escape strings probably has some negatives, but there can be helper functions for that.
- huon: I imagine it'll be easy to forget to call one of the helpers to escape a string
- cmr: Not sure about that.
- brson: cmr, seems like you have the go-ahead on this. Can you update the RFC with the whitespace changes, then let nmatsakis or brson know?

# UFCS

- nrc: the oldest RFC still not merged!
- nrc: Currently has the syntax from the March workweek -- single : in brackets to clarify the trait of the method getting called. - e.g., (X:T)::M(...)
- nrc: Either we should merge that, or nmatsakis wanted to do something else.
- nmatsakis: Yes, I wanted to write about it, but ahven't
- nmatsakis: I wanted to propose a different syntax which is easier to parse and not so ambiguous:

```
'<' Type 'as' TraitRef '>'::
<int as Clone>::clone()
(int: Clone)::clone()
Clone::<for int>::clone()
// This that would work now:
<[int] as Clone>::clone()
<&mut Foo as Clone>::clone()
```

- nmatsakis: I prefer this syntax because, in the version from the workweek, the parsing is much harder. You don't yet know whether this is an expression or a type.
- nmatsakis: We ignored it before, but it'd be a big pain. It'd work for paths, but not other kinds of types that are not paths.
- acrichto: How is this not ambiguous with the < operator?
- nmatsakis: It's binary, not unary; you can't put it as the first thing in the expression
- nmatsakis: Also, I like the keyword "as"
- bjz: Can this confuse the issue around static/dynamic dispatch?
- nmatsakis: Possibly. I think this is a corner case, syntax-wise, so I'm not that worried.
- erickt: how would this interact with type ascription (which we want an RFC for)
- nmatsakis: I think this works better than with a `:`
- nmatsakis: For example, the `int: Clone`, you have to hold off your parse for a really long time.
- nmatsakis: You can do a lot of the same examples with type ascription. But sometimes the value of self is not inferable from the return type (like size_of). But probably you would write:

```
T::size_of()
<T as SizeOf>::size_of()
[uint]::foo()
```

- nmatsakis: In that instance, type ascription just doesn't help: there's no type to ascribe to.
- bjz: can you put in a more complex type, leaving off the trait ascription?
- nmatsakis: I suppose we could make the ascription optional. [ed. see above]

```
[uint]::foo() // <-- unparsable
<[uint]>::foo() // <-- parsable but ambig in the trait, which we can accept
```

- pcwalton: I like that idea. Doesn't that allow us to do this kind of thing:

```
<T>::size_of()
<T>::default()
<T>::zero()    // instead of `{ let x: T = Zero::zero(); x }
```

- nmatsakis: yes, then we don't have an annoying path dependency thing.
- nmatsakis: It's not what you expect to write, but not that bad.
- nmatsakis: I'm fine with that; it's a simplification.
- brson: So this is how you call static methods at all?
- pcwalton: On type parameters.
- nmatsakis: You only need this when you can't infer the type of self.
- pcwalton: You might want to write T::default, like you'd write in C++. This <T> is like our version of the `typename` keyword.
- nmatsakis: It's consistent with how we provide type args to function calls.
- pcwalton: We'll still get questions about why T::size_of() doesn't work
- brson: What about in a UFCS world where you want to import a static method
- brson: Do you have to put a type parameter in the middle of the `use` statement?
- nmatsakis: yes, I think that is correct.

```
// UFS method import?
use <HashMap>::init;
```

- nmatsakis: This makes me think we want to keep the sugar for path names we had discussed.
- nmatsakis: We could still let you drop the `as Trait` if what you want it not a path.
- nmatsakis: the set of syntaxes would be:

```
<Type<...> as Trait<...>>::...
<Type<...>>::...::foo::<a, b, c>
Type::... // iff Type is a path
```

- kmc: too bad we don't have type params in modules.
- nmatsakis: we already have them on function items -- just as bad
- brson: What are we trying to resolve?
- nrc: whether we're happy to move forward with the proposed syntax, or any other issues with the original RFC?
- brson: It sounds like we're disinclined to use the old RFC at this point?
- nmatsakis: Should we close it an I'll write a new one?
- acrichto: we probably don't want to change and merge -- we should let it bake a bit.
- nmatsakis: I'll open a new RFC
- pnkfelix: (from chat) This requires whitespace in paths, which i don't like
- niko: Too bad (paraphrased)
- nrc: And we need to close the existing RFC.

# macro expansion in patterns

- kmc: uncontroversial RFC on this. wanted to check consensus
- brson: pauls approves
- nmatsakis: macros always controversial! but this is feature guarded, I suppose
- kmc: using them isn't feature guarded, but defining them is.
- kmc: There are already enough places where you can use macros that it seems weird to single this out.
- bjz: Allowing expansion more broadly would be very welcome. I can't write `concat_idents!` in a function identifier for example.
- nmatsakis: I don't see any particular problems here, over the known problems.
- brson: So this is an entire arm with branch head and expressions?
- kmc: no, just the patterns.
- brson: Does this add a new type of macro expression?
- kmc: metavariables? We already have one for patterns.
- kmc: I'm pretty sure you can do $x:pat already
- brson: OK, no controversy. I'm glad pauls weighed in. I'll merge it.
- nmatsakis: does this work in let statements?
- kmc: I haven't tried it, but it should.
- kmc: Should also work in function arguments, but that causes problems for rustdoc. So don't do that.

# unboxed closures

- pcwalton: I would like to implement unboxed closures. We've basically accepted this RFC.
- acrichto: we accepted an RFC for by-val captures; that's it.
- pcwalton: Didn't we accept strcat's unboxed closure RFC? (https://github.com/rust-lang/rfcs/pull/77)
- pcwalton: I would like to implement this RFC; it's needed for 1.0
- brson: Wait, you're saying you want to merge this RFC, but there's questions about the syntax?
- pcwalton: The RFC is incomplete. It doesn't describe the traits that you want (Fun, FunMut, FunOnce), which also impacts the syntax.
- nmatsakis: Also, bound region parameters.
- acrichto: It seems like we've agreed on some, but not all steps.
- pcwalton: But I want to implement it; it's needed for 1.0.
- pnkfelix:  (from chatbox): Wait, when did Unboxed Closures become *necessary* for 1.0?  Does "necessary" mean unfeature-guarded ?
- nmatsakis: I think we need more written down. We've discussed this, and some of the parameters are laid out, but there are many other considerations.
- pcwalton: But I want to implement before writing it up. There's no question we need it for 1.0
- acrichto: That's what pnkfelix was asking.
- nmatsakis: I thought we had a design that was forwards-compatible, but not fully implemented it
- pcwalton: I don't think it works; e.g. tupling up your arguments if they're DSTs.
- nmatsakis: We were relying on switching from a signature with an object type to a generic signature, and we're going to lose that ability.
- pcwalton: Exactly, so we need unboxed closures for 1.0
- pcwalton: The signature of the traits for unboxed closures is:

```
trait FnMut<A,R> {
    fn call(&mut self, args: A) -> R;
}
```

- pcwalton: So in order to have multiple arguments, you have to put them in a tuple, but you can't put DSTs in a tuple.
- pcwalton: Therefore you can't pass DSTs by value.
- nmatsakis: So without variadic generics (which we don't want right now), you can't pass DSTs by value.
- brson: It seems that all that strcat's RFC specified was to change to by-val captures, and you've implemented that, right?
- brson: But *this* RFC has not been approved.
- pcwalton: It's orthogonal, but by-val captures are silly without unboxed closures.
- nmatsakis: Everyone agrees about the direction.
- pcwalton: I'll just implement some syntax, which everyone will hate. We'll need a transitionary syntax, port everything to that, then decide on real syntax and port everything back.
- pnkfelix: (from chat) I have been trying to get clarity on what we are committing to for backwards-compatible support for 1.0
- pcwalton: I want to proceed on the implementation work that's needed regardless of the final design for unboxed closure.
- nmatsakis: I think there's value in thinking this through first, before doing a pile of implementation work.
- nmatsakis: We don't have to nail down the precise sigils or whatever.
- pcwalton: I'd like to do the implementation in parallel with the RFC.
- brson: On that note, the RFC says that this syntax will desugar to an unspeakable/anonymous type. Do we have a plan for that?
- pcwalton: Yes
- brson: Does it correspond to the RFC's plan?
- nmatsakis: No.
- brson: Here's my concern. The RFC does not match the plan.
- nmatsakis: I think we need to write the plan up. pcwalton, let's write it up.
- pcwalton: The RFC leaves a bunch of important stuff unspecified. We need to spec it out.
- brson: So you and nmatsakis are going to make sure that the RFCs match the plan.

# RFC 84

- nrc: RFC 84 is just a compiler optimization.
- nrc: We should close it and ask them to file an issue instead.
- brson: This is pretty clearly an internal implementation detail.
- zwarich: What happens if I try to store an Option<uint>?
- cmr: It should just expand to the next largest integer size. And for i64, just do what we do today
- brson: Isn't that just like putting two ints next to each other?
- zwarich: The whole point was to avoid the extra tag.
- luqman: I thought they included some way of specifying what you want to be the NULL
- zwarich: You still have to check dynamically, otherwise your language wouldn't be typesafe.
- brson: Can we close this RFC, which is an implementation detail?
- everyone: yes

