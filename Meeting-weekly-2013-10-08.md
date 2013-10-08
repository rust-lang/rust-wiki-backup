# Agenda 10/08/2013

- status update changes (brson)
- ~fn changes (brson) https://gist.github.com/brson/6781740
- Semantics of `rustpkg test`, as per https://github.com/mozilla/rust/issues/9003 (tjc)
- `rust` tool maintenance (tjc)
- code review on github is awful (acrichto)
- drop by value (acrichto)
- drop in static items (acrichto)
- hoedown (acrichto)
- fixing on osx 10.9 https://github.com/mozilla/rust/issues/6849 (acrichto)
- removal of crypto code (sha1 => siphash) (acrichto)
- Semantics of multi-crate packages, as per https://github.com/mozilla/rust/issues/7240 (tjc)
- Weekly triage emails (tjc)
  - email didn't seem to go out on 10/7. Should we offer to take over the script from Graydon, and implement a somewhat less ad-hoc reminder system?

# Attending

acrichto (A), arashed, brson (B), dherman (D), pnkfelix (F), kmc (K) nmatsakis (N), pcwalton (P), tjc (T)

# Admin

## Triage emails

- B: still on Graydon's machine b/c of security issues. Brian got a new machine that we'll use just for this, and triage script will run on that.

## How we talk about status updates

*Editor's note: for the present, the following section only applies to paid Mozilla staff who work on Rust*

- B: Remoties sometimes feel out-of-the-loop. Main concern is sometimes 2 people work on the same subsystem but don't communicate with each other. Want more effective communication method that's lightweight. Some people feel writing statuses takes up a lot of time/effort. 
- N: Lightweight thing at top of weekly agenda where each person notes what they're working on. Just written down in the Etherpad, people don't have to say it out loud in the meeting.
- B: I'm wary of adding new process without removing old process. It's just piling on more work for people every week.
- N: Should be something you do in 30 seconds when you log in.
- B: Starting next week, we'll add a status section and if people choose, they can add a very brief summary of what they're working on

## Code review
- alex: does anybody else think code review on github is terrible?
- alex: (missed)
- tjc: I think we could punt this.
- niko: think it's pretty bad, don't know what to do instead

# rustpkg test

- tjc: discussion about what 'rustpkg test' should do. #9003
https://github.com/mozilla/rust/issues/9003
- tjc: three possibilities. kimundi wrote up. now it encourages you to have test.rs file, a second crate root. could also have tests in-crate, or have test.rs as out of crate tests.
- tjc: is the alternate crate root the way?
- B: To be clear, alternate crate root is the same crate, but you have to re-declare all of the top-level `mod` things, plus have tests. in it. I imagine crates will get complicated the bigger they get, and this may not be scalable. test being another crate external to the main crate was what the intention was, I thought.
- tjc: not really a hack, but a part of the design that may not necessarily be useful
- B: Personally, I think it's best to put tests next to code. Alternative crate root would just be to redeclare the entire crate so that the tests that are next to code would actually execute. For me, it would just be the exact same file, which is repetitive.
- A: Big drawback of in-library tests is having to recompile the whole library. Both approaches have pros and cons.
- B: is it hard to support both?
- tjc: both the alt crate root and the test.rs method are supported.
- B: Can we support in-crate testing in a lightweight way?
- N: What is the advantage of test.rs? If you want to put tests for module x, put it in a subdirectory
- B: This would still imply that the tests are part of the same crate?
- N: Yes, it would just be a matter of whether you use --test
- B: That's already what happens
- N: So why do you have to recompile the whole crate?
...
- tjc: so maybe the only thing to do is documentation
- B: I would like it if it fell back to compiling the main crate with --test
- alex: i'm afraid that we're discouraging use of this mechanism that compiles out test cases when not building weth --test
- T: sounds like we should support all 3 "scenarios", with each falling back on other
- B: Personally I think the alternate crate root thing is weird. In my opinion, maybe don't document it.
- tjc: so make sure no test.rs works, document the other three ways?
- N: I think we shouldn't support the "repeat yourself" way
- T: Maybe I should just take out the "alternate crate root" way. Okay, I'll take it out and document the other two scenarios in #9003

# Removing crypto code

- A: We got a pull request to remove all crypto code from rustc, use siphash instead. However, siphash is not meant for preventing collisions. So what do we think? I'm OK with very common digests like SHA1 and MD5 with a disclaimer about not using them for serious crypto. I don't know if that's too much in the realm of us having to maintain crypto code.
- B: We informally decided to not implement crypto in Rust. Should we remove all hashes?
- P: We should ask a crypto expert.
- D: We chatted with Ecker (?) at the ACID1 party and he said "don't write crypto code?
- P: Do digests count?
- D: Talk to bsmith
- P: ok, we'll ask him if digests count as "scary crypto code". I don't think so
- K: Sometimes they can be scary. There are timing issues, even with digests. Still can have data-dependent memory access.
Conclusion: we'll ask the expert (Brian Smith). Alex will talk to him.

# `rust` tool

- tjc: there have been some bugs about the rust tool. it's orphaned. new people try to use it find it doesn't work. what should we do?
- B: It's the thing you would expect to use, but if new people try to use it and it doesn't work, it's a trap. It's also not in scope for 1.0. Maybe we should scrap it. 
- (many nods)
- B: Let's scrap it.
- T: I volunteer to go ahead and scrap it.

# Functions

- informal plan https://gist.github.com/brson/6781740
- N: We (Patrick, Brian and me) discussed functions. We came to some compromises and a plan we want to float. The idea: have 3 function-like types. The simplest one: bare fns, with no envt. Closures, with a borrowed envt. Procs: one-time functions with owned envts. Closures are callbacks that can access the stack contents, and procs are ~fns that are movable and have a fresh environment that captures by move/copy. Procs would only be callable one time each. This simplifies the current set of function types. We talked about having a keyword `proc` that would declare a proc instance: (see code below)
  - Bare functions
  - Closures (borrow environment)
  - Procs (one-time, owns environment)
- Syntax for creating a procedure:
```
proc { /* code, implicit upvars */ }
```
- Limit `do` syntax to procs:
```
do call { ... }
```
- Possibly update type syntax:
```
|T, U| -> R // closure
proc(T, U) -> R // procedure
fn(T, U) -> R // bare fn with Rust ABI
extern "C" fn(T, U) -> R // bare fn with explicit ABI
```
- N: We said that the `do` syntax would be used exclusively with `proc`s; this may be controversial. We also discussed the possibility of changing the syntax for function types. (see above)
- F: Motivation for limiting `do` to procs?
- N: Simplicity and clarity. Don't have to look at the callee to know whether you're copying or mutating values in-place.
- tjc: That would also go with my intution that I think `do` expects things to run only once
- N: Almost certainly, in those cases, you're creating stack closures, so `proc` might not be suitable. If you take a stack closure and call it once, you should write it using RAII. We also talked about a keyword to make RAII syntactically easier. `scope` keyword would allow making a dummy variable more easily. (See code below.) Or maybe with a macro.
Possible scope keyword:
```
scope expr { ... }
is syntactic sugar for
{ let _a = expr; ... }
or maybe we just use a macro
scoped!(expr { ... })
```
- P: I wanted to make sure that we can transition easily. strcat has argued that our current stack closures are suboptimal because they can't be used monomorphically like traits can. You can't take T:Closure(whatever) and return T. C++ lambdas did this. This really is an issue, and we should fix it. I don't think we need to fix it before 1.0, but we should make sure we can add it backwards-compatibly between 1.0 and 2.0.
- N: I agree with strcat; I think the way we do traits and object types is the perfect model in that it lets you choose between monomorphization or polymorphism. Closures are essentially similar to objects. If we extended object types with additional capabilities (the ones closure types already have), we could make closure types syntactic sugar.
- P: Our proposed closure type syntax doesn't have the & in it, so it's not a DST, and can't be made into a DST.
- N: Why is that a problem?
- P: Didn't we want something like fn `f<T:|A|->B> { ... }`?
- N: To unify closures and traits, I think we would wind up with
```
|T, U|:'r -> R 
to be syntactic sugar for
&'r mut Func:'r<(T,U), R>
```
- P: That's what I was thinking, So, the sugar for the bars ( | ) includes the & ? I was going to ask where the region goes. It's weird to have it afterward; I'd assumed it went before.
- N: There's a little bit more than a region -- it's the bounds in general. I had assumed:
`|T,U|:K -> R ==> K is 'r, Send, etc`
- N: K is a region bound or other built-in bounds
- P: I don't care where it goes, but I want to decide on something. Let's do what you [Niko] did.
- F: Only problem I can see is some people complained about -> R being our syntax for return, and if we put :K there, they might think it's the return type
- N: I think we won't be writing it often, but it does potentially confuse people
- P: Do we want to be able to say `fn f<T:|A|->B> { ... }` in Rust 2.0?
- N: Uh... sure.
- T: Is that higher-rank polymorphism?
- P: No, it's just sugar for:
`fn f<T:|A|:K->B> { ... }`
desugars to
`fn f<T:Func:K<A,B>> { ... }`
- P: strcat was going to make a Func trait and just impl it for closures. Will that mess anything up?
- B: This doesn't seem like something we'd "just go implement". It's a lang item?
- P: He's implementing it as a non-lang-item. I'm concerned about backwards compatibility
- N: As long as we designate it as mildly unstable, it's fine
- P: It's every single iterator
- N: Almost all code would continue to work.
- P: Let him implement it and we'll see what happens. (Can we make it a lang item in the future w/o breaking backwards compat?) Implement Rust-2.0-iterators in Rust 1.0 but hide them behind a flag.
- N: I'll have to think about it more, could be non-obvious consequences.
- B: Have we decided on a course of action for closures?
- N: Are there any concerns about or objections to this course of action?
- A: I'm not quite following everything, but it would be cool to write something up. 
- N: I started a blog post that's mostly done. I could publish it. It covers what I just said.
- A: That would be good. I'd like to have a little more detail written down.
- B: Is there already an issue for this?
- N: Yes, issue 2202 - https://github.com/mozilla/rust/issues/2202 . It's not specific to this plan. Maybe I'll make a new issue once this plan is better-documented.

# Hoedown

- A: We're using Sundown for markdown rendering, and it's no longer maintained. Everyone used it a year ago, and everyone still is. It's not broken. Hoedown is a v. recent fork of Sundown (the "Sundown of the future"). The maintainer of it contacted me to encourage that we use it. It fixed a few bugs, but nothing major. 
- T: If it ain't broke, why fix it?
- P: I agree
- B: Is github still using sundown?
- A: It's hard to tell.
- B: I think we should use exactly what github uses. Github *was* using sundown. Let's stick with it.
- A: I'll look into it and make sure what github is using.

# Fixing on OS X 10.9

- A: We're broken on 10.9 because of some linker flag we supply for the morestack function. Does anyone know why we have it? If you rm the linker flag, everything runs on 10.9, but I'm not sure if it breaks on 10.8 and earlier.
- B: If we remove no-compact-unwind, it fixes the problem?
- P: Has someone grepped the history to find out why it was introduced?
- F: The linker flag was added to address a problem w/ the unwind info for morestack. We're not testing segmented stacks now, so I'm worried if we remove the flag, something bad will happen
- B: OS X uses its own unwind info that's not DWARF. no-compact-unwind makes it use DWARF. If we omit the flag, segmented stacks won't work.
- A: Larger question: this totally breaks us on OS X 10.9
- P: Would be better to just not have segmented stacks at all on 10.9, but it's awful.
- B: We could just make everything else work on 10.9, then push the pain down the road when we rewrite segmented stacks
- A: No bot running 10.9 => no way to prevent regressions.
- K: we only want segmented stacks on 32-bit, is anyone running 10.9 on 32-bit hardware?
- P: 10.9 is only 64-bit
- B: we still plan to emit segmented stacks code even on 64-bit where the increment is huge
- K: Just because it makes the code simpler? It might be worth  not emitting that code at all
- P: AFAIK the plan on 64bit was "if you enter morestack, you're dead"
- A: Do we want the minimum stack size to be 1 page?
- P: Not enough time for this discussion
- A: Wanted to bring up 10.9 compatibility. 
- T: maybe just, for now, disable segmented stacks on 10.9? something > nothing
- P: I agree
- B: Great meeting!
