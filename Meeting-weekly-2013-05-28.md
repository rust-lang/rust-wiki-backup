# Agenda

* &mut pointers are noalias now - what are the implications? (brson)
* forbid "pub extern" and "pub impl"? (pcwalton)
* "unwrap" -> "get" (pcwalton)
* what should "Const" be called? (pcwalton)
* are we OK with going without select for a while? (pcwalton)
* 128i8 (pcwalton)

# `&mut` pointers are noalias

- N: I don't know the exact technical definition, but if the borrow checker is working correctly, - it should be true that an &mut should not alias any other writable pointer.
- N: There might be other readable pointers.
- N: We should figure out what it means exactly.
- N: There's also still a soundness problem around closures, which I'm still addressing.
- N: The recurring closure case, mentioned in blog post.
- N: Given that it's unsound, I wonder if LLVM will exploit this and maybe make more mysterious bugs.
- P: You'd have to make a y combinator manually, no?
- N: No, multiple closures that call each other recursively, or using conditions. Or something to do with closures making incompatible borrows.
- B: I'm worried about the impact of this on unsafe code. Any time you use unsafe to get a hold of a &mut, you have to make sure that you're not ...
- P: I think this is fine, you should not be making ...
- B: Not true. There's lots of code that makes lots of intermediate unsafe pointers and then finally produces a &mut
- P: From the point of view of the callee, there's no alias
- P: if you have a function that's taking 2 &mut pointers, if they could alias then your function would be wildly memory-unsafe. The only problem you could have ...
- P: Ok, I can see how it could cause a problem if you have a function that expects they won't alias, but the caller (using unsafe code) decides to pass in actually-aliasing pointers
- N: But that's bad anyways, noalias or otherwise
- N: I think we need a clear set of guidelines about unsafe code
- N: what your unsafe code has to conform to, what are the rules
- N: we might need to audit existing unsafe
- N: but I agree with patrick that if you're violating this you're probably already unsound
- N: (just reading what noalias means...)
- G: brian, did this back-and-forth answer your question?
- B: shrugging. not sure. I'm worried about scheduler code.
- N: any code in particular?
- B: nothing specific but I see it as a risk
- P: I really think we need this commonly
- B: changing APIs to * isn't the solution
- P: but they assume you're following the rules
- N: if the callee takes a &mut and you pass a *, that's not a problem unless you pass the same * in multiple arguments. If you do that, you're already doing something wrong. Similarly if you cast to a &mut in a callback, you're doing something wrong.
- B: ok, I understand what the caller's obligation is now, thanks.
- F: is this is just driving LLVM optimizations?
- G, N: just LLVM
- N: "pointer values based on arg1 can never alias pointer values pased on arg2", which is true up until you pass through a @. Probably fine for many cases, but I imagine you ...
- P: I think this means GEP, not load
- N: yeah, ok. that makes more sense.
- B: so this doesn't have any effect on @mut?
- N: there should be no effect. this gets back to .. you can't be writing to those values from 2 separate places at once. But you could be _reading_, that's what I meant by "there could be const aliases". So if LLVM is going to use this for CSE or something ...
- N: see what i'm saying?
- G: it might have decided it doesn't need to reload a value when it actually does due to a write
- N: yes
- N: the other thing worth considering is: do you just mean "no aliasing between arguments"? or .. what if I get a value from somewhere else, say a global value or something?
- N: That might help with other things. We should figure out what it means.
- N: Specific concerns:
     - scope of noalias. Does it only concern distinct arguments of the function?
     - Derived pointers means "GEP"
     - in other words, if I have a function `foo(a:&T, b:&@mut T)`, would noalias imply also that `a` and `*b` do not alias?
     - @mut, &const, closures bounds can all produce read-only pointers that may alias &mut
- G: if we just turned this on, and we are not sure if it means all hell breaks loose, is .. that currently happening? should we be turning it off again?
- P and N: back and forth about differences between noalias and restrict
- N: I think probably we can't pass &const and &mut
- N: also upvars and a couple others; 
- P: doesn't it _only_ affect multiple &mut pointers, not "other pointers that might be coming along?"
- N: That should be ok. Also `&mut T` and `&T`

# forbid `pub extern` and `pub impl`

- P: right now you can put `pub extern` to save yourself writing `pub` on each inner item.
- P: reads weird to me. How do people feel about it?
- P: changes the default.
- P: my specific concern with it is the same as `pub impl`, which is that when you're editing a long section of code, you don't remember / know whether each decl is explicit. You have to scroll up a lot.
- F: yeah, I was breaking mine into multiple extern blocks.
- P: as a result we forbade `pub impl` in the style guide
- P: why not remove from language?
- G: only reason I put this in was to minimize mandatory repetition
- N: probably don't want to be exporting a ton of C functions anyways. you want to write safe-making wrappers and such.
- _general ambivalence_
- P: `pub impl` seems worse, actually, since the function bodies expand the code between implied `pub` points
- P: does anyone object if I remove these?
- N: I think you should remove them both for consistency
- P: ok, cool

# `"unwrap"` -> `"get"`

- P: we have lots of functions called `unwrap`. You want to write `get`. But `get` copies-out.
- P: this is historical, probably. we probably want move-out the easiest thing to find, which means calling it `get`.
- P: most cases, it's the last time you're using the LHS, because you don't care about the error case.
- N: makes sense
- N: didn't we already do this, like on `option`?
- P: ok.
- G: nobody disagrees, let's carry on

# What should `Const` be called

- N: I think `Freeze` is fine
- N: Brian's argument that there are shareable things you don't want to freeze is valid
- G: I'm ok with freeze.

# are we ok going without `select` for a while?

- P: this is something probably Brian has an opinion on. I would like to jettison the old scheduler really soon.
- P: new sched is missing threading, but Brian has been doing lots of work on it, I expect it'll land soon.
- P: not much else missing. DNS (needed near-term), IPv6 (short-term ignorable), various failure modes (we can probably do something simple for now: child dies => you die, not much else).
- P: then there's `select`. Brian likely to object to my summary of how to make this work, but I think it's possible. I am wondering if we can live without `select` for a while just to help along the transition to the new scheduler.
- P: you can always simulate `select` with another task.
- P: we got rid of all the cases in servo
- N: I only care if servo cares
- G: how do you emulate it with another task?
- P: you have another task and it listens on ... 
- B: each port gets a task and forwards it to an either port.
- N: so not one task, but several.
- G: I don't quite get..
- N: one extra task per proxy you're wrapping things-you-are-waiting-on
- B: if the goal is to get rid of select, then yes, this will work; we'll eventually have a real thing for select.
- B: I'm less sure that we're close to being done
- B: the biggest missing-thing is split stacks. some of our tasks will stop working.
- G: it's all running on large stacks right now?
- B: yup
- B: still, there's nothing critical that depends on split stacks at the moment. We could still move to the new scheduler without that.
- N: I do think it'd be good to move asap, even if it implies some loss of functionality.
- P: yes, the old uv code in particular is a maintenance burden
- G: how close would this be to being able to start work on IO APIs
- B: no, we'd have to keep using `io::`
- P: 
- G: I know that there is no replacement io module right now, but I'm saying that we can start working on a replacement now, right?
- B: We have TCP done as a proof-of-concept
- G: If this were active as a part of the trunk, we could suggest to others in the community that they can follow this model and improve it?
- P: Already possible, actually
- G: How easy is it to activate the new scheduler?
- B: Just an environment variable
- G: So you can build newio stuff right now?
- B: Yes
- G: We should probably publicize this fact, there is lots of interest in better I/O support
- B: I'd like to get something out to the mailing list before interns all arrive about new scheduler, "what you can do to help"
- G: I wish I could say the same thing about the GC. Getting there.
- G: I don't have a strong feeling about when we do the cross-over, if it involves discarding split stacks temporarily, I think that's a judgement call that brson is best qualified to make
- G: Disabling an entire platform would be bad. I'd like to not lose 32 bit *entirely*. 
- G: We've got to be able to find the stack segments, that's a little bit of a trick, we'll have to have some conversations about stack segment enumeration and GC
- P: So the answer is basically "yes, we can live without select"
- G: Yes, I think so
- B: What about dropping pipes? If we remove select, we are a stones- throw away from removing pipes,
- P: Current pipes compiler would have to be rewritten for new sched anyhow
- B: I'm pretty convinced that pipe compiler is too complex for core
- G: I agree we're breaking up core libraries quite a bit anyway
- G: You'll have all the pieces required to build pipes available
- B: Yes, if someone wanted to maintain pipes, you could do it yourself, except for macros
- G: Note there is a PR on incoming for dynamically loading syntax extensions
- B: Preliminary
- G: Yes, but very cool.

# 128i8

- P: Currently we do a nice job of statically forbidding you from writing obviously out of range integer literals.
- P: Except that there is one value for every integer size class where it doesn't work, because the minus sign is not part of the literal.
- P: We have to conservatively accept 128i8 because it might be -128i8.
- P: There was a discussion about this on the mailing list that petered out eventually.
- tjc: Why not make it part of the literal?
- jc: What if it's a binary operator? 3 - 4?
- jc: This is why ML chose the twiddle (which I am not actually suggesting)
- P: Other possibility is just to accept everything in the lexer
- N: What about just making the lint pass smarter to distinguish 128i8 that appears under a `-` and those that do not?
- P: Yes.
- G: What do you do about range checking constant expressions in general?
- G: Since we are actually evaluating them
- N: I don't quite get it
- G: Do we allow general overflow in constant expressions?
- P: I like the "As if infinitely ranged" for constant expressions
- N: I had never consider that, but it makes a lot of sense
- P: My instinct tells me not to allow overflow
- N: I am wary of having two expression evaulation semantics 
- G: We've had people ask if we can have a mode where we turn on overflow checking on integer types at runtime
- G: I feel like possibly we can take our guidance from that, and say "if overflow checking is turned on for this module, it gets done, but not otherwise", but maybe we should warn anyhow?
- P: Maybe we can't decide that at this meeting
- G: It is part of the general const overhaul topic that we continue to defer


