# Agenda

- Goal sheet (Azita)
- Landing borrow checker (nmatsakis)
- Closure recursion (nmatsakis)
- typedef reform (pcwalton)
- removal of `<->` (pcwalton)
- `@` in patterns (pcwalton)
- servo build (Jack)

# Goal sheet

- *meta discussion about how to manage ourselves*

# Landing borrow checker

- niko: how should I land my new borrow checker?
- patrick: I usually patch bomb. Because of bors, things bounce many times. Little patches get in the way and cause breakage. Usually I end up with a set of patches then a bunch of smaller 'fix test' commits because of conflicts.
- graydon: Do you mean one big patch or one pull with lots of commits?
- niko: That's what I'm wondering. ....
- niko: What I'm asking is can I just push this and deal with the fallout?
< missed>
- niko: I usually go commit by commit, make a comment, then find it's corrected in a later commit.
- graydon: We're still in the stages of reworking major subsystems, and everyone is putting trust in you.
- niko: Ok, I'll just get it landed. What I've got is passing all the tests and building itself.

# Closure recursion

-  niko: Wrote a blog post. &fn is unsound. I don't think it's too important but does require a fix. Bottom line: we have to prevent closures from calling themselves.
- niko: It's very hard to actually call a closure, but it can be done. It's incompatible with a lot of things we're trying to do. e.g. 'for' loops need to borrow stack variables without worrying about creating hidden aliases.
- niko: I think it won't break any code
- niko: Until recently closures were limited to argument position... we expanded scope of closures without reasoning about what it meant.
- jclements: This will be a static error? You can fence out the Y combinator?
- niko: Yes. It amounts to making &fn affine.
- brson: Does this affect borrowed objects?
- niko: No, because their state is always in the self pointer which is already subject to the normal rules.
- niko: <missed> restructuring check so that upvars are treated like everything else.
- graydon: Reifying the stack and ensuring those upvars are treated like everything else.
- niko: Yes.
- niko: I won't get to that for a while but be aware of it.

# Changing typedefs

- patrick: Calling methods on typedefs, but you would not be able to `use` from them. Niko described a way to make `use` work as well.
- patrick: Easy for monomorphic paths, but with type parameters is pretty hard
- N: I think it is something we can scope in over time
- brson: Would be nice to do the types of things that bjz is trying to do on the pull request
- niko: Putting the type parameters near the place where they are declared is common.
- graydon: I don't understand from the comments whether bjz was satisfied with the workarounds

# Removing swap

- patrick: Saw a comment that we can't remove swap, for some reason I can't remember.
- patrick: I do know swapping array elements will not borrow check if we remove it.
- niko: We can have a function for it
- patrick: Often swap isn't what you want. You want n-ary swap, 'shuffle', taking a few elements and reorder them
- patrick: Treemap wants this.
- tjc: Reason before had to do with purity, so not an issue now - https://github.com/mozilla/rust/issues/3466
- niko: We can probably get rid of swap
- niko: One thing that would be useful here and elsewhere is a set of operations that can divvy up a &mut [T] slice into disjoint subslices or into pointers to disjoint elements. 
- pnkfelix: I think I needed swap at some point
- niko: I think due to bugs in the borrow checker
- pnkfelix: let's wait till that lands then
- niko: that'd be helpful, it's hard to deal with the overlap between old and new
- graydon: I would very much favor doing anything to get borrow checker changes done soon, since they really affect everyone
- graydon: kind of like resolve
- pcwalton: new checker should make people's lives easier, particular around flow sensitivity

# `@` in patterns

- P: I know we discussed in Vancouver but don't recall the outcome
- P: Can we remove `x @ Pattern`?
- pnkfelix: What about `x = Pattern`?
- jc: right now expressions can appear in grammars
- N: didn't we say we just want constants?
<missed>
- jc: maybe I should make a small proposal on email and people can agree or bikeshed
- graydon: I think in any case, `=` is not going to be used elsewhere in the pattern language
- niko: It's bad enough that we have to deal with guards that mutate state

# Servo breakage

- jack: Servo doesn't build because of an ICE
- patrick: Tim may want to look into this. ICE in pattern matching in trans. #6117
- patrick: `match` forgot about one of the patterns (or something) and tried to index into an array
- patrick: It only happens on one platform in monomorphised code.
- graydon: Which platform?
- patrick: Mac Mountain Lion
- jack: So it doesn't affect any rust developers but affects people trying to use rust
...
- patrick: Call to resolve::iter.
- niko: Looks like the bind_irrefutable_patterns code? Anyway let's talk about this offline.
