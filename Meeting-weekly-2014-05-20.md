# Agenda 5/20/2014
* trait-bounded return types (aturon) https://github.com/mozilla/rust/issues/11455
* String/&str (acrichto)
* P<T> in the AST (acrichto) https://github.com/mozilla/rust/pull/13316
* Moving repo to rust-lang (brson) https://github.com/mozilla/rust/issues/14003
* RFC  https://github.com/rust-lang/rfcs/pull/65 (nrc) change : to = (or something else) in struct literals (anyone else keen on this, or should I just close it?)
* RFC  https://github.com/rust-lang/rfcs/pull/79 (nrc or anyone else) undefined layout of structs (its new-ish but doesn't look like it needs a tonne more discussion)
* RFC https://github.com/rust-lang/rfcs/pull/73 (nrc) effect system (can we close this)
* removal of ~str/~[T] (pcwalton)
* by-move upvar captures (pcwalton)
* removal of cross-borrowing (pcwalton)

# Attending

cmr, luqman, zwarich, erickt, nrc, acrichto, pcwalton, bjz, brson, pnkfelix, niko, aturon, azita, jack, huon

# Status

- cmr: heavily refactoring the lexer
- brson: vpc, contracts, docs, minor 1.0 issues
- acrichto: facade work, Arc<Unsafe<T>>, various 1.0 things
- nrc - DST
- pcwalton: removal of ~str/~[T]
- Luqman: building compiler-rt with asan runtime, updating llvm for nonnull attribute
- nmatsakis: working on trait resolution, sp
- pnkfelix: control-flow graph, makefile hacking
- aturon: library guidelines, Path papercuts, RFC for "impl T"
- erickt: v4 libserialize, game tech meetup planning

# Action Items

- brson: Ask github about 'rust' org, and move repo, also 'servo' org
- pcwalton: update the RFC with String and str
- acrichto: Merge String/str RFC
- brson: close RFC PR 73
- brson: merge RFC 79
- brson: close RFC PR 65
- nrc: create new RFC to fix expr grammar in block heads
- pcwalton: get strcat's closure RFC to reflect desired plan
- brson: set up all meetings for afternoon time
- acrichto: merge RFC 82
nmatsakis: close RFC 81

# Repo location

- brson: I want to move repo to rust-lang instead of Mozilla. Not sure when is the best time but sooner seems better than later. OK?
- cmr: github has redirects
- acrichto: let's announce and give warning
- brson: concerns?
- bjz: "rust" or "rust-lang"
- acrichto: do we even want to check for "rust"?
- brson: I'll send some feelers out. Not that big a deal.

# UFCS

- nmatsakis: I'm going to amend RFC, I'll post it, I'm working on this

# Trait-bounded return types

- aturon: I'm going to try and write an RFC and then we can talk about it after that

# String vs str

- acrichto: Lots of debate. I think we have reached point where we have two factions and just have to decide. Seems to me that the majority of the community is behind String and str. No way to please everyone. Just have to decide, merge RFC saying: "String" and "str".
- brson: nrc, do you want to say your piece?
- nrc: I agree that there is no way to please everyone. I also agree that there seems to be a majority in favor of String and str. I guess we take it. I still think it's wrong. (ed - tempted to insert some invective but nrc was infallibly polite)
- acrichto: pcwalton, can you update RFC?

# Effect system RFC

- nrc: This seems so far beyond post 1.0 I don't think we should keep it open.
- nmatsakis: I agree with you. 
- nrc: Not that it's necessarily a bad idea in the future but it seems very far away. 
- nmatsakis: We had a tag for this, 'postponed', maybe we can use that.

# Undefined layout of structs

- nrc: Proposal not to guarantee layout without #[repr] annotation. Hasn't been up for too long. But discussion seems to have come to a close fairly quickly. 
- nrc: does this need more discussion?
- brson: Is there any kind of transition plan?
- cmr: I think we extend the ffi-safe lint to include structs not marked with this attribute pretty easily
- brson: Is that part of the RFC?
- huon: It is in there
- acrichto: it is a bit more complex than current enum lint, you'd have to in theory deeply visit all types in the FFI...
- nmatsakis: Why is it different from enums? Seems the same.
- acrichto: our types in FFI lint is not very good
- acrichto: <something about C++???>
- nmatsakis: you're saying in C/C++ there's a one-to-one layout and that's useful because you can control it?
- acrichto: the default is different for us
- nmatsakis: usually, they just leave everything undefined and rely on convention
- nmatsakis: it seems reasonable. It seems like enum/struct are two sides of the same coin.
- brson: sounds like no controversy here. let's do it.

# Struct literal syntax

- nrc: I think we should use `=` instead of `:`, since `:` is all about types and this is not a type
- nrc: Opinion seems to be split
- nmatsakis: I don't share your aesthetic dislike for it. There was something like, if we had an ascription operator, there would be an ambiguity, but that is true for `=` as well. I'm inclined not to take action.
- brson: I think we can easily punt on it, and looks wishy-washy if we make a change at this point. We get enough flack for other changes.
all: Close it
- nrc: Comments about the way we parse struct literals in a for loop: we just search for a `:` which seems quite fragile. If we want to change that, not backwards compatible
- nmatsakis: the idea is to define a subset of expressions where the termination is an open brace?
- brson: what is the change with these expressions
- nmatsakis: right now, we can accept something like:

```
// first struct literal is ambiguous until parsing the ':'
if Foo { x: 22 } == Foo { x: 22} {
    ...
}
// Fix would be to parenthesize:
if (Foo { x: 22 } == Foo { x: 22}) {
}
if cond { ... }
```

- nmatsakis: the problem is, you don't know if it's a struct literal until you get to the `:`. Proposal: in those places where we dropped parens from the C standard, we don't accept struct literals.
- nmatsakis: You'd say that the first case is illegal. You'd have to write it with parens.
- brson: how is the grammar that lets you do that defined?
- nmatsakis: You'd have a subset of expressions -- the main expression nonterminal, and a sequence of other nonterminals for precedence. The if nonterminal etc. would exclude struct literals.
- nmatsakis: Or maybe just two branches.
- brson: Sounds like this could help cmr with parsing work.
- pnkfelix: Would this help with macros, reading a bunch of tokens until curly open brace?
- nmatsakis: Maybe. We don't have a real grammar. nrc, it'd be helpful to write up a grammar for this (but we'd first have to figure out what ours is ;)
- nmatsakis: Do we need a single hierarchy or two branches? struct literals right now are an atom at the bottom, but we'd want to remove that. Not clear.
- nrc: I'm not sure how that interacts with the [missed] stuff. I think that with an abstract grammar it would just fall out.
- brson: so, nrc will continue exploring fixing this issue with the grammar?
- nrc: sure, I guess that should be another RFC
- nrc: So I'll close the RFC and start a new one about the grammar.
- brson: Yes.

# Removing ~str

- pcwalton: I've been removing ~str from the libraries, but I'm worried that stuff is going to creep in over time.
- pcwalton: we're carrying a lot of legacy stuff in the compiler. i'd like to remove this -- make it an error -- until we have DST
- pcwalton: there is currently no method of generating ~str or ~[T] outside of unsafe code, but there are various places where we have pattern matching on ~str and equality etc...
- nmatsakis: hard to object to removing code that destructs values you can't create.
- acrichto: you'll have to say `.as_slice()`
- pcwalton: I don't think any of this code will be useful because it's the wrong representation.
- nrc: My local queue changes all this code to work with the DST approach.
- pcwalton: Can I remove from the parser?
- nrc: yes, I don't care about that.
- pcwalton: It would be good to remove soon, though; don't want dead code in trans.
- nrc: Landed by the end of the week.
- pcwalton: OK, will remove from parser but keep everywhere else.

# P<T> in the AST

- acrichto: eddyb has a pull request replacing @T with an owned pointer called P<T>, under the hood represented by a ~T.
- acrichto: I also have a patch to replace with Gc<T>.
- acrichto: Have other people looked at this patch?
- nmatsakis: I scanned it.
- acrichto: Not sure about the long-term AST direction. Is this heading the right way?
- nmatsakis: I'm not happy about an unsafe AST map.
- nmatsakis: The benefit is that you can do in-place folding, which is nifty.
- nmatsakis: This is not exactly the direction I had imagined for the AST, but not necessarily a bad direction. It seems that in general an owned tree is more friendly for parallelization.
- brson: Can we ever get lifetimes to work out for a safe AST?
- nmatsakis: probably not with in-place folding. Bottom line, I thought this was an interesting patch but I just don't know if it's what we want.
- acrichto: There's a *lot* of unsafe code.
- brson: Are there performance wins?
- acrichto: I assume so, but I haven't seen measurements.
- pcwalton: Why not use Rc<T>?
- acrichto: I just wanted to decide on this PR; we'd want to do it soon to get rid of @T.
- cmr: the first version of the patch used Rc<T>, but didn't seem necessary.
- pcwalton: Needed for safety.
- nmatsakis: Could just copy data out of the AST map, rather than unsafe map.
- pcwalton: Would be nice to keep rustc safe.
- brson: There's a lot of hacks in the compiler -- why pile more on?
- nmatsakis: I'd ultimately prefer an array -- just index it, and then you have your AST node. Plus compact memory layout. This patch isn't heading in that direction.
- nrc: BTW, eddyb is commenting on IRC right now.
[ everyone reads ]
- huon: in theory, all the folding is done before we map the AST.
- eddyb (on IRC): ast_map being unsafe is provisory. unsafe is needed because of issue #13703
- acrichto: We could take it as-is, but switched ownership to Gc.
- acrichto: The refactoring would be kept, but under the hood it would be safe until we can address this issue.
- nmatsakis: We'd lose the in-place update for now, but keep all the hard work making it possible. I prefer that.
- acrichto: I can talk with eddyb on IRC about that and see if we can reach conclusion.
- brson: sounds good.

# by-value (move) upvar captures

- pcwalton: I would like to propose doing by value upvar captures, with reborrowing of &mut
- pcwalton: for 95% of closures, this is fine.
- pcwalton: it doesn't lose any expressive power from what we have today, since you can take explicit refs
- pcwalton: the size of upvars is generally small. 98% are 64 bytes or fewer. These numbers are true for both rust and servo -- there is very little discrepancy between them.
- nmatsakis: I was dubious earlier, but no longer.
- nrc: Is this just strcat's proposal?
- pcwalton: minus the capability of labeling upvars, which doesn't seem necessary
- pcwalton: what i'd like to do is make all closure upvars by-value, which will do all the backwards incompatible stuff right now. that will future-proof us for unboxed closures, modulo a few kinks. we expect very little code breakage after that.
- bjz: how does this relate to &uniq?
- pcwalton: &uniq will be removed from borrow checker.
- pcwalton: also dropping proc, in a later RFC
- pcwalton: reborrowing of &mut will be important, have to work with nmatsakis about that.
- brson: surprisingly short conversation about closures!
- pcwalton: data makes conversations short
- brson: pcwalton: please help get the RFC in shape.

# Removal of "cross-borrowing"

- pcwalton: This is about "cross-borrowing", the conversion of Box<T> to &T in favor of deref.
- pcwalton: That's the only remaining coercion that auto-borrows. It also works with Gc which is weird.
- pcwalton The final smart pointers hard-coded into the language.
- pcwalton: Generally, a kind of weird, warty thing at this point. I'd like to remove it. Perhaps in the future we can provide a generic form that you can use for your own smart pointers.
[ general agreement]
- nrc: What would code look like in the future?
- acrichton: This definitely wants an RFC
- pcwalton: agreed, just wanted to get initial reactions.

```
    fn f<T>(x: &T) { ... }
    
    let x = box 3;
    ... f(&*x); ...
    
    fn g<T>(x: &mut T) { ... }
    
    let y = box 3;
    g(&mut *y);
```

- acrichton: How are you going to do coercions of trait objects?
- nmatsakis: Oh yes, that's a problem. Without DST, no way to write it.
- nmatsakis: You could have a function that takes &Box<T> and unsafely transmutes to &
- nrc: How will DST fix this?
- pcwalton: You'll be able to write &*
- acrichton: If I have &Trait, I can't deref it, because Trait is not a type. DST would fix that.
- brson: nrc, did that make sense?
- nrc: Not sure.
- brson: It sounds like you have the go-ahead for an RFC, for sure.
- pcwalton: I'll also gather some data.
- nrc; My feeling is a possible half-way thing would be not to do it for &mut
- huon: what about &mut Trait?
- nrc: For the convenience of not having to write &* all over the pllace, have autoborrowing for anything that implements deref.
- nmatsakis: I would like some convenience here, but I'm not sure that's the design I want. Various valid use cases, not sure how to reconcile them all.  Rc<T> and you want an &T, or maybe an &Rc<T>. I'd like to start from a clean slate and gather data on what's most common.
- acrichton: Opposite of pcwalton's suggestion of removing all automatic magic here. But this is all RFC discussion.
- pcwalton: I just want a consistent system. If we start arguing over whether this is a good idea in general and get ourselves deadlocked because of that (and there will be people who object) -- we should be consistent. Right now we don't have this because of Vec and StrBuf. We should get it consistent now, and then evolve it in a consistent way.

# quickcheck

- brson: refresh us on the status
- huon: originally a Haskell library for generating test data and reducing failures to a minimal example.
- huon: burntsushi wrote a library for this. suggestion to merge into main repo with a syntacic extension to use as the test runner.
- brson: the last time we discussed, we were worried about merging prematurely
- acrichton: not clear how widely used this would be. There aren't problems with it, but in comparson to say the regex trait, not obvious how widely used it would be.
- brson: in general, we want to let community crates bake and gain maturity before merging.
- brson: regex was a special case.
- huon: the problem is the lack of a package manager, which makes it hard to use community libs. maybe we should wait for cargo.
- bjz: how useful would quickcheck be for testing libstd?
- acrichton: currently, not used at all in the Rust distribution. we'd like to see some test usage before bringing it in.
- nmatsakis: isn't there a chicken-and-egg issue there?
- acrichton: Not necessarily.
- pcwalton: I'm a little worried about the fit for rust here. It's a direct port from Haskell, which means it's using type signatures in a "clever" way that may not be ideal for Rust
- nmatsakis: I wouldn't mind trying it out. I'm currently struggling with a hashmap bug, maybe I can use it there and see if it helps.
- cmr: If it was useful for the standard library, wouldn't it be useful for everybody?
- pcwalton: not just about whether it's useful, but whether it's ubiquitous.
- acrichton: ultimately, you should just be able to point cargo at a git repo
- brson: we don't have a migration path into cargo -- we don't want people to be able to rely on availability and then not be able to guarantee it later.
- brson: General consensus: not comfortable merging quickcheck yet.
- huon: I'll close the PR and submit any patches back to burntsushi.

# Meeting time

- nrc: Can we make this time the permanent time for the meeting?
- pnkfelix: At this point, I've blocked off this time slot. Better to keep it consistent.
- nmatsakis: The time isn't ideal, but I can live with it.
- brson: OK, so we're getting rid of the other meeting time, and always do it at this time?
- all: Agreed

# Drawback section placement for RFCs

- cmr: PR 82. Can someone merge that?
- acrichton: Moving the drawback section below the details section.
- brson: acrichton, can you merge that?
- acrichton: Yes.

# Tail calls

- bjz: strcat's PR #81
- nmatsakis: an RFC from strcat to guarantee tail calls now that LLVM provides support. That would be awesome, but we should do it later.
- nmatsakis: we have a huge queue of things, we should delay on this.
- brson: niko, can you close it?
- nmatsakis: yes.
- pcwalton: Close as "postponed"?
- nmatsakis: yes.
- erickt: for something like that, how do we respond if somebody in the community offers to implement?
- nmatsakis: even if someone else does it, it's not "free": it takes energy to review and maintain it. The total energy involved over 1.0-1.5-2.0 for that is too great for the payoff here.
- nmatsakis: This is a perfect example of a postponable feature. Not to trivialize it -- it's awesome.
- Erickt: Can we make a list on the wiki to publicize this?
- brson: we do have a tag for postponed, right?
- acrichton: Yes. We should tag RFCs with that, close them, and then there's a record.

# Virtual struct RFCs

- nmatsakis: Can we postpone RFCs related to virtual structs?
- nrc: I thought they were high enough priority that we want to do them.
- pcwalton: servo hasn't been clamoring as much for them, but we're not worried about getting Bobby Holly's time right now -- we have so many other things to do.
- metajack: the spidermonkey PR is there right now. josh would know more.
- nmatsakis: maybe we should leave them open. I can read them again.
- nrc: It seems like 1.0, we want to check that we're not closing the doors backwards-compatibility wise. So we should check for that, even if we decide not to implement something. Might want to decide what we ultimately want to implement.
- nmatsakis: alright.
