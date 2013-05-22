## Agenda
* Rust is super great (seconded)
* procs, fns, oh my (nmatsakis)
* Owned -> Send (pcwalton)
* ptr::null() and cast::transmute() into the prelude? (pcwalton)
* removal of let foo = 1, bar = 2 (pcwalton)
* @ in patterns -> ??? (pcwalton)
* #[inline(maybe)] and curry! macro #6616 #6634

## Attending

graydon, nmatsakis, jclements/bus, jack, pcwalton, brson, tikue, eatkinson, azita, tjc

## Let it be known: rust is super great!

- N: proposed
- B: seconded
- N: resolved, moving on

## Closure types, see niko's blog:

http://smallcultfollowing.com/babysteps/blog/2013/05/14/procedures/

- N: instead of having sigils, we'd have fn which is a stack-closure and proc which takes ownership.
- N: couple of design tweaks we can talk about 'proc'. does the basic idea work?
- N: I think we need to do something like this in order to nicely handle the dynamically sized types proposal.
- G: what happens to @fns ?
- N: that comes back to the matter of proc. they're subsumed by the proc type. Depending on how we chose to handle the defaults, you could say that just saying "proc" will cause copy-into-environment, but doesn't constrain the types of things it copies-in. Then @proc (say) would be the equivalent of @fn.
- P: Part of this observation is that most of the time, @fns are things that ought to be @Traits. We almost always want to close over data shared between multiple functions. The only exception really is rustglut
- N: Not sure what rustglut is doing but it's probably a C callback and it should probably be an @Object or @proc
- P: We could probably region-ify it and avoid GC anyways, which would be better given its uses.
- N: The proc type would have inside of it an ~ pointer that points to an environment that it owns.
- N: The proc type could then be put in an @ box or ~ box or whatever, as currently
- G: can you elaborate on the "necessity" of this for dynamic-sized types?
- N: we'd have to write "&mut fn" otherwise
- N: functions have to be linearly-tracked
- N: we can't recurse with the same environment, currently. in order to prevent recursion, we have to track it and prevent free-aliasing, and that's what &mut enforces. so where we now write &fn, that is not able to enforce the requisite safety properties.
- N: I was going to solve this by saying "&fn is tracked differently than &anythingelse" but that's highly dissatisfying.
- G: so this is really folding "&mut fn" into "fn" just for brevity
- N: yes
- N: the equivalent of the major 3 types that we use today look like:

```
&fn(S) becomes fn(S) -> T
@fn(S) -> T becomes @proc(S) (or else a trait)
~fn(S) -> T becomes proc:Owned(S) -> T
```

- P: niko and I came to almost the same design. the basic issue is that ... I don't think we'd need proc except that the difference between capture-by-ref and by-val is important sometimes.
- P: now that niko has brought it up, the fact that &fn captures by ref and ~fn captures by val is kinda weird in the first place, now that I think of it.
- P: the fact that this split fixes memory safety is just a nice additional outcome
- B: what about once procs?
- N: not sure what use there is for non-once procs
- B: I need them for the event loop. they have fns that have to fire multiple times
- N: maybe, maybe it's best to keep 'once' present in both.
- P: yeah, I guess we're trying to be too minimalist?
- B: so once fns have to live in mutable slots?
- B: like a boxed once fn is useless
- N: yes, that's true
- N: you have to pass them around linearly
- G: somewhat similar to the old `prog` notion we had way back, which were the spawnable-things. we switched to spawnable functions when we added destructors.

```
[once] fn:'r K(S) -> T
[once] proc:'r K(S) -> T
```
where `K` are builtin traits: `Owned`, `Const`, `Copy`?

- N: what name do people want? some people suggest `proc` means doesn't return a value
- P: proc is ok with me
- G: I'm fine with it
- N: what do we do about inference / notation? Do we use the ||-notation? I've wanted a different notation
- B: do you have a suggestion?
- N: 

```
proc |a, b| expr
do spawn proc { ... }
```

- P: do we want to continue to infer sigils?
- N: that's what we're talking about, really. There won't be sigils. So this is about inferring proc-ness
- P: ok, so what's the signature of spawn?

```
fn spawn(p: once proc:Owned())
```

- P: oh. so you're not going to have sigils?
- N: yeah
- P: that's nice. cleans up all the questions about  ..
- B: that looks worse to me. the sigils were clearer
- P: I think it looks better but we can maybe infer proc-ness
- P: possible compromise: do spawn { ... } infers, || does not infer
- N: I'm ok with that. keeps the complications in the code, but nicer UI
- B: we could also use macros?
- P: I think there are a lot of other spawn-like do uses in the scheduler, no?
- B: yes
- P: and the inference doesn't work right when you overuse ||, no?
- N: that's only one, I have to resolve all the compilations anyway
- N: I think we could fix them
- N: I'm not so worried about the technical side
- N: I'm more concerned about what gives the reader the most information
- N: I'm concerned that I don't know what a function takes, so when I see a `do` I don't know what it's taking, it's hard to read and know what happens to the block
- N: that's why I was arguing for explicitness 
- B: yeah, true
- P: yeah, I guess. if it's `do spawn proc` people might complain that it's not as tidy as `go <expr>` but ...
- N: there is always a macro for terseness
- P: maybe. spawn has quite a number of small tweak names
- B: the macro form is most important for giving the spawn a name based on source coordinates
- P: oh yes, good point.
- N: it seems that we ...

## macro import/export

- N: aside: are we going to have macro import/export? 
- P: I talked to dave about how we might drop it before 1.0 
- B: seems like we really want it. people keep proposing new ones.
- P: yeah, just proposing ways we might do 1.0 w/o macros

## back to dynamic sized types
- G: this was only thing blocking dynamic-sized-types proposal?
- N: yeah.
- B: ok, so this doesn't make all pointers thin though, right?
- N: no, not all. & on dynamic sized remain fat; everything else thin.
- N: life's nicer for the GC and such
- G: ok

## Owned -> Send

- P: there was some discussion about Owned being the wrong word, moving back to Send, for this kind bound.
- P: We really care about Send
- B: RC-pointers. They're explicitly not-owned because we want them to be not-sent.
- P: Whereas ARCs are Owned
- P: Really what we're talking about is sendability
- J: didn't it used to be Send and we changed it? Why?
- B: Ownership is part of the type system, Sendability was a weird thing to mention in the type system
- B: But this concept, as it turns out, is only really used in concurrent code
- N: And as an annotation, NotSendable is much more readable than NotOwned
- B: Can we change Const to Freeze while we're at it?
- G: ... I don't think I know what this means
- G: when would I put this bound on a thing?
- N: when you want to know something can-be-frozen
- B: it has to do with concurrency, yes? To know you can put something in a shared-readable location.
- N: it has to do with that, and is used as a hack to prevent cycles in RC as well
- G: ok
- B: sounds like agreement, move on?

## ptr::null() and cast::transmute()

- P: can we put these in the prelude? all my code uses them
- B: that shouldn't be the case, though, right? I mean ..
- P: this is the sort of code that people write in rust though. we do. others do.
- <commentary about patterns of awkward of *>
- B: making transmute easier to call seems unfortunate to me. you can do anything with it. it's such a big hammer.
- P: how about null at least? it's harmless
- P: the zero trait perhaps?
- B: depends on what you call it
- N: ok, perhaps
- P: I think of the zero trait as go's idea of zero-values. empty-initializers and such.
- N: nicely solves the vec-blanking problem?
- P: we could put `zero`, the function, in the prelude, from the `Zero` trait
- P: ok, don't put `ptr::null` in the prelude, put `zero` in the prelude, resolved!
- N: I want to add a plea here: there are a variety of transmute-like functions that are much-more-narrow casts. you may want to use them. many times when I see crashing code, it's because of a transmute that's been too ambitious.
- B: yes. I'd argue for even more narrow transmutes.
- N: maybe we should try to make enough to cover each case, and remove transmute itself.

## removal of `let foo = 1, bar = 2;`

- P: did we actually have consensus to remove this?
- <crickets>
- G: wanted to make sure: this has come up before, but is the person who might have objected just not present today, and we're sneaking this past them?
- N: I think it was punted last time it came up in concern for what I might have thought
- G: ok! onwards, cut it.

## subpattern binding: for, as, @, =, something else?

    match foo {
        Some(bar @ @Baz(0)) => ()
    }
    match foo {
        Some(bar for @Baz(0)) => ()
    }
    match foo {
        Some(bar as @Baz(0)) => ()
    }
    In clojure:
    match foo {
        Some(@Baz(0) as bar) => )_
    }

- P: if we use `as`, it'd prevent us from having cast expressions in patterns, which we use for `'c' as u8` but that's because we don't have 'c'_u8 or some other kind of explicit literal number suffix on characters
- P: if we do `as`, maybe `<pattern> as <pattern>` a la haskell
- G: also we do sometimes (often?) discuss removing `as` for casts
- P: out of time, sleep on it, come back next week
