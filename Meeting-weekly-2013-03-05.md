# Agenda

- Hash out work week agenda (nmatsakis)
- Object types and the associated trait (#5087) (nmatsakis)
- Parsing: Generate code from grammar, or grammar from code? (pnkfelix)
- Priority-preference: rustpkg vs. GC? (graydon)
- Acceptability of nominate+milestone strategy for triage (graydon)
- removal of "copy"
- Ord/Cmp/TotalOrd naming
- time permitting: &&
- time permitting: explicit enum discriminants

# Attending

brson, graydon, tjc, azita, jclements, nmatsakis, pcwalton, pnkfelix

# Work week agenda

- nmatsakis: traits
- N: I just wanted to throw out a few things to talk about.
- traits, not very well-defined parts of spec. What kind of rules to put on the coherence check, etc.
- P: functional deps?
- N: revise design around static types etc.
- N: complex issue, 
- G: can you bring _examples_ for each of these, written up before hand? Arriving at a meeting about this where we're just learning the issue means it's hard to decide, yeah.
- N: yes, I can prepare examples/presentation for this.
Backward compatibility: not as important, but we'll discuss it. To make associated types work, we may want to change name resolution.  We'll talk about it then.
- P: +1 for in-person. Hard to do async. 
- A: as we come up with list, keep in mind that Niko and Felix are going to be in PJs for first day, how to break it down etc., make sure that things don't overlap.
- N: +1. Scheduling is important.
- N: other things?
- P: depends on what states regions are in, lingering things there? eg: random borrowck errors? May want a discussion about it. Also possible that your changes will resolve this.
- N: play it by ear.
- P: also, about trait hierarchies... don't care that much.
- G: I'd like to do a day of "what goes into 1.0" planning. And what is "we are not going to even try for that until a later language rev". At least a broad set of topics. We did this before and have tried a couple other times and I think it likely to move quickly and concisely in person.
- N,P: +1. Love to have a good idea of what 1.0 is.
- F: Backward compatibility is an issue?
- N: We may want to do *some* things that are backward compatible.
- P: Breaking some things is fine.
- N: ok.
- N: P has been pushing Rust toward having backward compatibility (stability?)
- G: I'd like to spend a little time discussing constants. This doesn't need everyone.
- N: G says constants, me too.
- P: especially for associated constants, but not for 1.0
- G: also some time discussing the linking model and metadata model. inter-crate dependencies. mostly just writing-down what we all believe to be true facts and realizing which of those surprise one another.
- G: Maybe some discussion of core/std library contents (and their traits, and maybe principles for design or inclusion/exclusion). again not so crucial that everyone be there. just a way to soak up a day on something useful.
- N: +1
- General agreement with everything G says.

# Object types

- N: opened bug recently, want to make change if possible, will simplify borrowck. Right now, if you have an object type, like:
- N: If you have an object type @T, ~T, &T where T is a trait
- N: It sometimes implements T and sometimes doesn't
- N: This has to do whether it not uses the `Self` type
- N: Example where it is unsafe:
```
trait Eq {
    fn eq(&self, other: &Self) -> bool
}
```
- N: Can't use this with objects like `@Eq` because we don't know the true type
- N: So can't guarantee that `other` will have the same type   
- N: So. I'd like to say that object types *never* implement their trait. You can implement it yourself if you like.  sidesteps complex specifications, lets the typechecker do its job. How do people feel? Maybe could provide #[auto_deriving].
- B: ok with this.
- G: I am ok with this. Object types and trait bounds are pretty different.
- N: trying it out, let you know if problems arise.
- P: might simplify trans a lot.
- N: that was one of my motivations. started fixing other things, found sketchy code in trans.
- P: Causes segfaults.
- G: One caveat / question: is there a clear notion of when a trait T can be turned into each sort of object type @T ~T &T ?
- N: The nice thing about this proposal is that we sort of don't have to write such a rule, it's just that some methods won't be invocable.
- N: The last time we talked about this I was holding on to the idea that sometimes @Trait would implement Trait.  That introduces various complications.  But if we make this rule, we always know when the receiver is an object type and when it is not.
- N: what is going on: Eq is not useless. It's useless as an object type.  Eq is one of the types for which it's not useful to use it as an object type.  We used to have special rules, but we won't need them after this change, because we won't be able to invoke the methods. Think about Java's .equal() method.
- B: what is the rule, again?
- ...confusion about what the rule is...
- N: Today we have this rule already, and so I neglected to mention it, but it is important background:
  - you cannot invoke a.b() where a an object instance
  - if the type of b() references `Self` as a parameter or return type
- B: so only specific methods can't be called?
- N: correct, we also have extra rules today which pertain to when an object @T implements T
  - but those would go away
  - the reason for those extra rules is to prevent this kind of example:

     fn foo<A: Eq>(x: A, y: A) -> bool { x.eq(&y) }
     ...
     fn bar(x: @Eq, y: @Eq) -> bool { foo(x, y) }

  - This is of course illegal under my proposal as well.
- B: also need rules about other situations...
- N: yes, right. didn't mention those, various other restrictions.

    @self method must be with an @Trait object
    ~self method must be with a ~Trait object
    by-value self can't be used with any object
    &self can be used with @Trait, ~Trait, or &Trait,
    (must also consider mutability)
    basically analogous to the rules that borrow checker would normally enforce
    for borrowing @T as &T and so on

- G: why can't by-value self be used with @T?
- N: because it copies/moves the receiver also we don't know what the receiver's type is.
- N: maybe it could be used with ~T, not sure
- N: for example if the receiver is an integer vs a struct it would be get passed differently according to the calling convention but we don't know that until runtime
- N: and we don't know its size. I think it can't be done.
- G: I am not understanding, I think. fn foo(@self, ...) can't be invoked?
- N: that is not by-value self (or not what I meant by by-value self).  That can be invoked on an @Trait but not ~Trait
- G: Ah, ok.
- N: I am referring to fn(self, ...)
- G: got it.
- N: seems like agreement.
- all: tacit agreement.

# Parser

- F: I've been curious about parser stuff
- F: It will be non-trivial to port the parser as it is today to a standard grammar
- F: Various things like obsolete checks etc
- F: Do people think we will go with hand-written parser and automatic extraction of grammar
- F: Or base the parser from a grammar and generate it by some means
- N: I would prefer generating in the long term but I don't have a strong opinion
- JC: I also support the notion of a parser generated from a grammar
- JC: I didn't know you were working on this, maybe we should touch base
- F: Only a litle, I am more interested in generalized LL/Earley grammars and modern parsing technologies
- JC: Let me suggest a general direction: it is my hope that Rust will have balanced delimiters, which is pretty vital for making macros work, so I'm imagining a grammar where for instance delimiters are always balanced ...
- F: ...some sort of parenthesized grammar, makes sense not sure if it's the right answer but is feasible
- JC: I'm trying to update token trees so that they ...
- G: for newcomers to the project: I have always grumpily insisted that we try to keep the lexical grammar regular and the CFG LL(1).
- F: It's more that I was curious which approach was more likely to bear fruit in the long term.  I didn't want to look into a parser generator if we are not going to use it.
- F: Well, I don't know if the grammar is LL(1) today
- F: Certainly the parser as implemented is doing more than 1 token of lookahead
- N: Not necessarily inherent to the grammar
- G: "doing more than 1 token of lookahead" does not mean it actually has a backtracking point there; it might just be factoring (afaik it usually is)
- F: My personal interest would be to hack on a parser grammar that accepted any grammar in any case.  But that's more of a side project.
- G: I would like an LL(1) confirmation and auto-generated parser for rust. This is something I've wanted to be able to guarantee before we call something 1.0. Whether that drives the compiler is ... less certain. But I'd want it to be existing-and-maintained at least. Which makes it worth wondering whether to drive the full compiler from it.
- N: +1 to that, though I don't care too much about LL(1) vs e.g. LALR(1), LL(2), etc
- F: Regarding the "existing and maintained" aspect, I imagine that if you had a formal grammar you could at least check that the code bases parses with that grammar, whether or not we're actually using it in rustc itself
- G: it's always _nice_ to be able to run a compiler off a generated parser. I worry only that I've never seen a production compiler for which this is true. I'm sorta curious why. Usually people say this has to do with compatibility / performance / error recovery?
- JC: It seems to me that if you have a grammar you should use it
- N: I think that's perhaps confined to C++? I'm not sure how widely true that is
- PCW: Perl, PHP...
- N: ...maybe Python?
- G: ok. I am not at _all_ opposed to meeting this if it's possible. I very much prefer formal over informal.
- *general agreement that it would be nice* :)

# Priority preference: rustpkg vs GC?

- G: Sure. Minor q: shall I spend this week making the GC go faster / work better or hacking rustpkg to do something useful? 0.6 will (currently) ship without a working package manager.
- P: I vote GC, i think it's more important. 
- N: +1
- P: rustpkg doesn't add technical debt, just makefiles.
- G: (surprised face, but ok!)
- P: okay... GC is really important.
- N: need them both, can see case either way.
- JC: +1 GC.

# Acceptability of nominate/milestone strategy for 1.0

- G: ok, second minor Q is I'm thinking of asking the mailing list also, ... is it ok to essentially mass-un-milestone everything and work off a system of nominate-for-milestone / weekly meeting to clear out nominations with either accept-for-milestone or reject-throw-back-into-bug-pile. Trying to make milestones _useful_ for something (they are presently useless)
- P: I guess so, I don't have a strong opinion here, except that I agree that milestones are currently useless
- N: +1
- B: Would this mean that milestone issues have actual "meaning"? Like, are we expected to finish them?
- G: yes, milestone acceptance would mean "I think we can actually meet this".
- T: My opinion is that they might be more meaningful if we had smaller scale milestones.  As it is I kind of dumped a lot of stuff there when it seemed a long way off.  
- P: I think the idea of dumping to the next milestone is what's causing them to be useless, they are getting too full. I feel like we should have a planning meeting and decide what the milestones are, and then just take them off if we can't meet them.
- T: We can't just stop having milestones because there are too many bugs, and we need some way to prioritize.
- P: Isn't that what triage is for?
- B: It sounds like graydon is suggested we do triage once a week...
- G: yes, I'm suggesting a structure for doing a weekly triage meeting that accepts / rejects / nominations and edits-down the milestone list.
- P: If we actually went through all the bugs it'd take hours
- T: the first one would take four hours but we don't have to cover all the bugs all the time?
- P: I guess so but I'm not personally focused on fixing bugs right now and don't expect to be
- T: It might also help volunteers to know what to work on
- G: we currently use bug tracker for 2 things (keeping track of stuff, period, and supposedly-but-not-really prioritizing). I want the second thing to come into focus.
- N: Graydon---clarification, you didn't quite say we should review all open bugs, right? More that people would nominate things of interest?
- G: I did not sugest us doing a triage meeting that covers all open bugs, no. We don't currently have available schedule for that, I think.
- G: it'd be great if we could but I don't think I would be able to convince all of you to attend such a meeting!
- G: because it would last a week or so
- N: I feel like it's useful to have feedback on what problems people find important, if we're going to be fixing bugs might as well be one that people care about
- G: yes, that's the point of "nominate". it's a todo list for a triage meeting that is less-than the entire set of bugs ever.
- Azita: can we just setup a meeting and see how it goes?  See how far we can get doing triage in an hour?
- P: I don't know if I'd even attend to be honest, right now
- B: He was actually suggesting that we integrate a brief nomination type of thing into the weekly meeting, right?
- Azita: My concern is that because it's been so long we couldn't even do the first one within this meeting
- G: "triage of all the existing bugs" is a _different_ topic. it's something I'll discuss separately. I'm talking about nomination-triage as a way to make milestones meaningful. Only. Just "make milestones meaningful". there's a separate triage-everything-in-the-backlog problem.
- P: My concern is that we usually run out of time for these meetings and weekly triaging will take some time.
- T: Yeah it should probably be a different meeting, with a time limit
- F: Yes, a per bug time limit
- ...
- T: I feel like we need to have more triage than once a release
- P: Sure, I agree with that, maybe there is a happy medium time "once a month"?
- P: the project is immature enough that not all of us are fixing bugs, the value depends on how often we will be fixing smaller bugs
- G: I concur with patrick to some extent that there's a maturation threshold involved.
- P: it's easy for us to say at this stage "it'll eat your laundry" but once we want to ship something close to 1.0 we'll have to get more serious about this
- T: But how are we going to get from 1,020 bugs to 0 in 9 months?
- G: yes. we need to get to "an understanding of them all" and "none that we've accepted as blocking that milestone". I'm not -- _not_ -- discussing the 1000-bug backlog here. I'm discussing a strategy for nominate-and-accept. The backlog is a different topic. We will clear out the backlog separately. Ok.
- Azita: do you need to get to zero?
- JC: No that's totally unreaslistic
- P: I guess my point is that I don't see a point of spending time on triage unless we're actually actively working on fixing those bugs.
- P: Maybe we can bring this discussion to IRC.
- G: ok.
- 10 minutes: next topic!

# Removal of copy (pcwalton)

- P: I've brought this up on IRC a few times
- P: I propose removing `copy` keyword
- P: There is this current subset of types which are copyable only via the copy keyword
- P: e.g. ~T, types that would require malloc to copy but which are still copyable
- P: This doesn't seem worth building in to the language
- P: We can use a method `clone()` and `#[deriving_clone]`
- P: Which then also allows custom behavior (important for ARC)
- P: Type parameters like `<A:Copy>` would become `<A:Clone>`
- P: #[deriving_clone] is implemented
- P: How do people feel about this?
- T: Copy seems like a relic of an older era
- G: +1
- N: +1
- N: I'd like if it were called `Copy` and not `Clone` someday.
- P: Well, what about something like `struct Point { x: uint, y: uint }`?
- P: I was envisioning that `Copy` would be things that are implicitly copyable today
- P: ...and then there would be a rule `impl<T: Copy> Clone for T`, but this does violate the coherence rules today... maybe we can fix that in some way...
- N: I guess I don't mind `#[deriving_clone]` on structs, we derive other things too
- P: the nice thing is that it eliminates `Copy` kind, so we'd be down to `Const` and `Owned`
- JC: Would this ... eliminate types being copied by default?
- P: No, just not part of the trait grammar
- N: When you're writing a generic function, you will want to use the clone() method because generic values won't be implicitly copyable
- P: Is there general consensus?
- N: It seems like there is general consensus on the basic idea, maybe we'll hammer out Copy vs Copy/Clone at a later time

## TotalOrd, Ord

- P: strcat introduced TotalOrd like Haskell's Ord
- P: I think we should rename TotalOrd to Ord and rename Ord to Cmp
- P: Where Cmp is used for partial equality and the < opeator etc
- P: Usually Ord is the one you want anyhow
- JC: What is the difference ?
- P: For background, TotalOrd is a trait with a single function that returns an enum Less, Equal, or Greater.  This needs to be separate from Cmp because Cmp actually defines the `<` operator, which has funny semantics around NaN (Unordered)
- P: Alternative name for Cmp is Rel
- N: +1
- G: fine by me
- N: Weren't there two traits where this was relevant?
- P: Just Ord, you don't need it for Eq, you can just use Ord for Eq
- N: Basically <T:Ord+Eq> is an anti-pattern as would be <T:Ord+Cmp>
- P: Yes, should be <T:Ord>
- B: Does Ord inherit from Eq or Cmp?
- P: No
- B: Aren't they kind of the same thing?
- P: Totally distinct operations for some types, partial vs total
- B: Isn't equality total?
- P: Not with NaN
- N: I've been doing some research into how other languages handle this, every single one has some sort of hack and inconsistency to deal with this, pretty similar to what we're doing here ultimately (e.g., in Java, Double.equals() is not the same as "a == b" if a and b are doubles, etc)
- G: Yes. Do not get creative here. Research carefully!
- P: I also confirmed with codec people that it is important that the basic operators use IEEE floating point with its normal nan semantics for hardware performance
- G: think we're done? Thanks for coming!


