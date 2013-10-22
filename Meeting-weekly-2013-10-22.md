# Agenda 10/22/2013
- drop by value (acrichto)
- segmented stacks (acrichto)
- priv keyword, #8122 (acrichto)
- crypto update, #9744 (acrichto)
- Improving the volunteer experience (tjc):
  - VMs that contributors can log in to to debug cross-platform build failures?
  - volunteer participation in meetings?
- Tim's next project (tjc):
    - ideas: field inheritance; splitting libextra; linking; checked arithmetic; move optimization; resolve overhaul
- gate/remove GC completely? (brson) https://github.com/mozilla/rust/pull/9923
- mut self/~self (brson) https://github.com/mozilla/rust/pull/9989
- ~Any (brson) https://github.com/mozilla/rust/pull/9967

# Status
- acrichto - landed morestack, landing rt improvements, putting uv in its own crate, bug fixes
- tjc - landed rustpkg date fixes; OPW; landed "find crates in $CWD"; worked on C dependencies
- brson - intern project descriptions, 1.0 roadmap
- pnkfelix - talking DST with niko

## crypto removal
- acrichto: got a PR to remove crypto stuff. Feedback from the security folks is not that we're doing things terribly badly right now, but they pointed out that bugs in things like SHA hashing can lurk for years (they did in Firefox). Maybe not remove them entirely?
- brson: that PR was just to remove the hash functions?
- acrichto: yes, removed all but setHash. I think we should leave the SHA variants but remove MD5, etc. Compiler uses SHA1. It's in libextra. Flag it as not-crypto-suitable. We'll maintain for bugfixes but not security fixes.
- brson: that sounds good.

## tim's next project
- tjc: rustpkg has some work to be done, but wrapping up. Most interesting / important (listed above in ideas). Opinions on what's important from that group? Field inheritance vs. splitting libextra vs. linking vs. checked arithmetic vs. move optimization...
- brson: nmatsakis opinion?
- acrichto: do all the things!
- nmatsakis: NOT move optimization, as I'm working on it. They all sound good. How dire is field inheritance?
- tjc: Need to figure out how we want it to work first. Resisting it for now. brson said it's desired for servo...
- nmatsakis: working on a document describing it:
- nmatsakis: draft here https://gist.github.com/nikomatsakis/7104501
- nmatsakis: some of the tasks are smaller than others. Maybe hit a small one first? libextra seems smaller.
- tjc: part of it is setting up the pkg & continuous integration infrastructure work.
- acrichto: linking might be interesting. It's not certain what needs to happen.
- nmatsakis: what does that mean?
- brson: native library problems. Everywhere we specify linkargs, we need to figure out how to specify that better
- acrichto: and static linking doesn't work at all
- brson: hard-coded platform-specific libraries also seems bad
- nmatsakis: can we do better? has anyone done better?
- tjc: can at least have datatypes instead of strings for managing them
- acrichto: when we use linkargs, it's almost exclusively for listing a library. Also some weak symbols and whenever we produce a binary we add a bunch of platform/linker-specific path args. Probably just need "link this library in" as everything else is compiler-specific.
- nmatsakis: is the library name platform-independent?
- brson: not always, no
- acrichto: all our C++ code brings in like 10 different libraries, each with different names. BUt it's the standard C++ ugliness for all platforms. Compilers.
- brson: and GL
- lbergstrom: servo, too. tons of android/x/OSX libraries are different on each platform
- tjc: linking sounds shorter than rustpkg, so maybe that first
- pnkfelix: mingw->llvm tools on Windows?
- tjc: not I!
- acrichto: there's no headway on building or seeing if that stuff works. strcat & cmr said they were interested. Might have gotten static linking working, but we have no headway yet. That's definitely the mingw migration path
- pnkfelix: I bring it up because maybe if we're LLVM everywhere, does that clear up some of this stuff? Or does the platform-specific stuff never go away?
- brson: still uncomfortable with a string representing all of our linking info
- tjc: not a waste to do that kind of clean-up, even if we switch entirely to LLVM after that. Still need the work afterwards.
- brson: sounds like linkage first
- nmatsakis: let's define the goals more concretely first
- tjc: there's an issue with some stuff written down. I'll flesh it out and either talk about it at next week's meeting or send mail.

## mut self vs. mut ~self
- brson: expand the grammar to allow these to be mutable?
- pcwalton: +1
- tjc: seems consistent
- nmatsakis: makes sense
- acrichto: patterns?
- pcwalton: still can't have mut & mut self
- tjc: nobody needs to do that...
- pcwalton: you can't reseat mut self. you could assign the whole thing, but that's just assigning every field

## drop by value
- acrichto: drop takes &mut self but self by value instead?
- pcwalton: proposal - easiest way is to require Drop be static and destructure its components in the argument list. If you take Drop by value, naively, it will infinite-loop unless you took care to destructure yourself. Empty Drop will just Drop and Drop and... so, require that you destructure yourself. If we just require they do it in the parameter list, that does it syntatically trivially. 
- acrichto: the first thing most stuff in the runtime does is call a method on itself. That makes destructuring painful.
- pcwalton: then we need to figure out how to fix infinite loop
- nmatsakis: previous plan: disallow self to be moved in its entirely only in the Drop function. Basically, special rules in Drop?
- pcwalton: seems like a really ugly rule
- tjc: it's a non-type-based rule, which is suspicious...
- pcwalton: seems like a cure worse than the disease
- brson: is this important enough to require a special rule?
- nmatsakis: if we go away from zeroing things out, does that make things less safe?
- acrichto: I don't know of any destructors where we move out of self
- nmatsakis: send messages in destructor? You can also swap an Option. Maybe it's not worth allowing it, as you can just use a Cell.
- acrichto: This keeps feeling like something we want. &mut self is useful for not-unsafely doing things, but the weirdness from self byvalue seems to outweight the benefits.
- nmatsakis: I'm persuaded
- brson: less is more
- nmatsakis: Keep it &mut self.
- pcwalton: Drop2 is byvalue self!
- nmatsakis: the option workaround is easy enough
- acrichto: I'll find the issues and comment that we are not changing for 1.0

## ~Any
- brson: 2 things in PR. Propagates failure message. Also propagates ~Any. Is that (second thing) what we want to do?
- tjc: seems like the right thing to do. Was implementing the same thing in rustpkg tests using a string, which felt really ad-hoc just because you're going across tasks. This type seems to be the only way to deal with the problem. If you're using try, it's nice to get more than a string.
- acrichto: looks like a send_str or a ~Any
- pcwalton: how is ~Any implemented?
- acritchto: It's a trait.
- pcwalton: That sounds right. We can refine it later.
- acrichto: Something like this for errors in I/O as well. 
- brson: Maybe an error string?
- acrichto: Implemented unfortunately. Can't have a bunch of default methods. Need those and then tons of extension traits.
- nmatsakis: will get much better when DST work goes forward, at least from my quick glance at it. ~Any makes sense. I would not want the result type *always* produce ~Any. Is that what's being proposed?
- brson: I've considered it
- nmatsakis: if what we're talking about is:
```
    type IOResult<T> = Result<T, ~Any>;
```
- pcwalton: It's very ML'ish (exns)
- nmatsakis: what does this address?
- pcwalton: Same that gives rise to monad transformers in Haskell. A function that calls I/O and then calls the database. Then you have some I/O work and then talks to the database. So, database.rs might have an error. And you want this function itself to return a result. So, you need the wrapped function to be able to transformed error into either I/O error or the database error
- nmatsakis: or you have ~Any
- pcwalton: which also requires a transformer
- nmatsakis: That case seems to be of libraries working together. vs. smaller-scale. Type inference produces error messages with structure in them. But they could use a separate type instead of ```Result``` with no great pain.
- brson: consensus is move in this direction? Needs further refinement, though.
- acrichto: We want something like this, so we might as well merge it now.

## volunteer experience
- tjc: Came out of toronto summit. Theme was treating volunteers as 1st class citizens. How do / don't we do that? 1. Volunteers not in meeting or triage (there is some admin stuff we can't share outside MoCo). 2. Some volunteers have been blocked on trybot access or log in to a bot with a different platform to debug on their non-desktop platform
- tjc: the second seems like something we could help with. try access is split off thanks to brson. But fixing access to other platforms seems like something we could help with.
- brson: access to try is really useful, but not clear what we can do there. It requires a signed agreement (which we've been asking form). Confirmation on those takes time. How can we improve it? releng is not a big fan of it (though Firefox does it too, but they keep try bots separate from production bots).
- tjc: is that why releng was afraid?
- brson: mainly that we need to be making sure there are no viruses, etc. getting loaded onto our bots
- tjc: sandbox? But that requires effort to set up. Kind of nice to help people debug stuff somehow. Maybe make VMs available? But this costs time to set up.
- brson: setting up separate try bots is very hard - we can't reproduce the environments on all bots because of all the configuration. Maybe cloud bots. We have Windows & BSD captured as AWS AMIs. Not identical Macs. We do have four of them, which are never all active. Lots of effort, though, and we have a lot of automation to do already.
- tjc: maybe on "think about later list?"
- azita: maybe talk to releng about the right way to do it before we do anything.
- tjc: if they know how to do it for Firefox, maybe do it for us, too? But it's probably a cost.
- azita: releng is already panicking a bit about all the stuff we have opened on them because there is too much.
- nmatsakis: anything better to improve the volunteer experience? Meetings?
- tjc: keeping meetings to a small group is nice, but want to be inclusive...
- azita: not in this meeting. Maybe a monthly rust meeting?
- brson: unclear just what we would discuss in this meeting
- lbergstrom: probably e-easy actually being easy would be bigger, given our OPW experience
- nmatsakis: make sure the bugs have enough background would probably help more. Bugs are often more a "note to self"
- pnkfelix: triage should be doing that -- don't just bump but add background
- tjc: try to make sure that E-Easy has descriptions.
- nmatsakis: w.r.t. triage, updating the examples would help. It's not done all the time. Particularly helpful for beginner who don't have the background knowledge.
- lbergstrom: just more info on Easy bugs.
- brson: anything concrete?
- tjc: try to have details on Easy bugs in triage & make sure they're easy

## automated memory managment
- pcwalton: TODAY, we have two things. @ and RC. Neither check for cycles. Both handle mutability identically. They do basically the same thing. RC is in a library and requires .Clone. 
- brson: cycles are destroyed with @-boxes....
- nmatsakis: on task termination.
- pcwalton: RC needs to do that as well. brson and I discussed just removing @ entirely and just using RC, since they're so close. And we don't want @ in the future anyway. Since we lose nothing over today, why not just remove it and add GC later? It's not removing GC from the rust language - we never had it implemented anyway.
- nmatsakis: Depends on your definiton of GC. No tracing GC.
- pcwalton: The question is how we add it seriously.
- tjc: How did RC end up having a totally different implementation anyway? Because it was in a library?
- acrichto: yes. Nothing in the compiler itself uses RC today. Just @. We don't have a lot of experience playing around with it...
- pnkfelix: We should do the exercise of porting rustc to use RC. 
- pcwalton: have to
- pnkfelix: There was a proposal to feature-gate it.
- pcwalton: porting rustc to use RC, we get the experience. Just feature-gating is a weird way of saying "@ is not for you"
- nmatsakis: advantage of @ is that you don't need clone. Smoother syntactically to go from RC to GC where you write the types. But the code... I guess clone doesn't harm.
- pcwalton: just annoying
- pnkfelix: Copy kind/trait/constructor?
- nmatsakis: we worked hard to remove it
- acrichto: Are we killing the @-syntax? I think we're basically killing GC and RC completely because they're so wordy and annoying to use. No auto-deref, having to clone/borrow/new all the time means we're basically steering everybody away from those types of memory.
- pcwalton: Not terribly persuasive. We have to make smart pointers easy to use. They have to be easier to use. Just making @ better doesn't help - we have custom smart pointers everywhere in servo. The solution to that is making custom smart pointers easier to use.
- nmatsakis: what order makes sense?
- pcwalton: I'm fine with rustc suffering some.
- nmatsakis: It's mainly porting costs for us and others. Deprecating @ would work, at least in its current behavior, especially if we are bringing it back in the future.
- nmatsakis: adding something like a typedef: `type GC<T> = @T`
- pcwalton: commits us to having gc
- nmatsakis: Who does *not* want to have GC?
- pcwalton: Means somebody has to write it by 1.0.
- nmatsakis: Wouldn't be a bad thing to have it, and do we need it for 1.0?
- pnkfelix: I think we need it, as much as I don't want to write, I will
- pcwalton: can feature gate it then. We're then committed to have all the conveniences of borrowing.
- nmatsakis: auto-deref?
- pcwalton: and methods and stuff like that.
- nmatsakis: We are committed to that, in any case. It's just not tenable to deliver the poor syntax. When you work in servo you need those things, right?
- pcwalton: yes.
- brson: Don't remove yet until we have a path to library GC. Do we want to put on the feature gate?
- pcwalton: maybe the typdef. The problem with the feature gate is methods. @self. What nmatsakis said is we could add a typedef and feature-gate which allows forward compat. but doesn't work for methods.
- nmatsakis: how?
- pcwalton: can't do GC self today. We could just fix that and then feature-gate it.
- nmatsakis: We're going to need this library solution anyway. Don't have all the ingredients for full forward-compatible syntax yet. Maybe that's the goal? Could be the 1.0 goal. Agree with pnkfelix that we sort of need to have a working GC for 1.0.
- pcwalton: For 1.0, we could have GC be magic. Feature-gate @, tell them they have to have the standard library to use it.
- nmatsakis: Say we plan on having GC but it's not ready yet? 
- pnkfelix: So 3rd parties couldn't use it?
- brson: We have to add so much that we'd be very close to having it all.
- pcwalton: Except skip writing a GC
- nmatsakis: Pcwalton is saying set up the syntax but keep our bad ref-counted implementation and fix that later.
- brson. Decision on this PR.
- pcwalton: I say no.
- nmatsakis: Feature-gating @? 9923? I think of feature-gating more as a warning like "unstable/unsupported", so it's a clear signal to our users.
- pcwalton: That does sound reasonable. That feature-gates are warnings, not a "internals-only" thing.
- brson: that's a decision!
