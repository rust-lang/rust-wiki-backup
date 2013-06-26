## Agenda

- 0.7 (brson)
- @mut (pcwalton)
- iterators

## Attending

attending: jclements, tkuehn, dherman, eston, azita, brson, toddaaro, ecr, bblum, eatkinson, pcwalton, sully, felix, jack, tjc

## 0.7


- azita: graydon's out, wants us to review. need to make decisions about what to do with open issues, etc.
- brson: best release position ever, we look good to release, don't worry about open bugs. Some key pulls still need to get in
- pcwalton: +1
- bblum: +1
- p: get your changes in. if they're break-the-world changes, let's discuss them, though bors will probably keep them out. don't know of any outstanding ... ?
- brson: I'd like to see Send-to-Owned and Const-to-Freeze
- p: it's blocked on one failing test, one that sully added, that'll be today
- b: I like ffi, but may not happen. niko is working on that
- b: I have a serious bug with stack memory usage.  many tests xfailed because of stack size
- p: we should talk to niko about ffi stuff, he can offload onto me. I want to get rid of copy, but ffi stuff is more important for 0.7.
- b : not a lot of changes. 
- p: can we switch to new scheduler?
- b: not in the tree. no one has reviewed it.
- b: bounced in and out for a month. anyone who wants to look at it is welcome. only about 6K lines of code.
- p: I want to close the tree until it lands
- b: not ready yet. task killing doesn't work, 2% of ...
- dave herman: graydon had this idea of audits--accounting? so you could get info about how much memory is being used, set limits, etc.
- b: not so useful/relevant any more, with exchange heap. It's too expensive.
- p: same with task killing. many of those features are going to have to go away for performance reasons
- dave: blog post from "atomic". They claimed "you can't leak memory." this struck me as bizarre.
- all: they probably just are noticing gc.
- b: back to 0.7. not a lot of language changes, but lots of library stuff. most of the work from community members. anyone want to write docs for rlnotes, or... 
- p: more commits than ever.
- b: lots of activity
- p: mostly libraries, also perf improvements, memory usage. compiler not quite as embarrassing.



## @mut 

- p: there's been some discussion about @mut not pulling its weight. whenever people use it, they get dynamic borrow-check errors, all the time. after talking it over with strcat & niko etc., maybe the right thing is to remove @mut, and use explicit cells to do dynamic borrowing, or introduce a new type called slot, which does dynamic borrowing. by making it explicit, you make it obvious what's going to happen. two things.

- 1) it becomes obvious when it can fail. you have to scope it (or not, with dtors)
- 2) we can explicitly document it in the library type. when you use it, you'll probably read the docs. right now it,s too easy, people are surprised by fails.
- 3) you can choose where the borrow happens on a more fine-grained level. you don't have to borrow a whole structures at once. eliminates some gotchas. niko & i went back and forth... this sidesteps the problem by making it explicit in all cases. this hase been a very abstract thing, hard to illustrate. 

- dave : thoughts; 1) looks like null pointers vs. option types: do you want convenience or explicitness? with this one, the lack of safety outweighs convenience... this comes from real experience. What will this look like in real code if we make this switch.  I think it may look one way on paper, but it might look different when we really try it.  
- p: maybe we should try it first, before making a decision
- d: just find a few files, try it out, see how painful it is. If it works, expand it
- bblum: doubly-linked data structures are the key example here. 
- d: wait... what?
- bblum: right. it's just a good case to look at.
- tikue: you already need options to implement a linked list
- d: options are bad... you can prove through some ad-hoc reasoning that it's going to be some, but the compiler isn't smart enough, then you end up with wasted code, or put in comments that say " I used get here because it's safe", but most people still agree they're useful
- p: null pointers, for all their evils, are a known and understood error. not like dynamic borrow errors. they're unfamiliar, because it's an affine type problem. so maybe we can make it more explicit.
- d: failures become more pedestrian.
- p: you're altering the mode of this thing; it can be in immutable or mutable mode. 
- d: library naming can help with that. it becomes less of a language concept, more of a library concept
- p: ...
- p: the naming is going to be tricky, there.
- d: you tried to freeze it twice?
- p: no, borrow twice.
- d: still seems like it's generall easier for people to swallow API protocols than language changes.
- p: this is giving up on trying to get mutable borrowing right inside @ pointers.
- bb: I like the idea of moving it to a library. it's easier to grasp that a given library has a pattern that may fail, than than the language has a weird feature that may fail. my PL background prefers this solution. I like the idea of using some kind of option thing to make it clear when you're allowed to access the thing or not. 
- p: there's a nice advantage to moving it to a library: you can have a library call that explicitly handles what is now the borrow-check fail. 
- j: could be complicated
- p: i don't expect anyone to use this...
- b: if we remove this and *mut, then the only mut will be &mut. can we get rid of this?
- p: probably not, we need this for ... mutating stuff.
- b: can't hoist out into all slots?
- bb: no, doesn't fit with existing type system
- p: i think &mut plays a very special rule in the borrow-checker. it knows deeply about &mut, that it's affine, that it's the only owner, and so it can track unique paths, just as it can with unique pointers. wouldn't work as a library. 
  - `&` - aliasable, non-owning, immutable
  - `&mut` - non-aliasable, non-owning, mutable
  - `~` - non-aliasable, owning, mutable
- p: all distinct and important rules. &mut is the only way to have something that's non-aliasable but mutable.
- ...discussion about what's the most awesome thing about &mut....
- p: even to make slot work, it's critical that we have this. it has to be aliasable and non-mutable.
- p: I can talk about iterators
- felix: I needed the concrete version of this, so... could you send out mail on this
- p: not sure whether to piggyback on cell, or build something else
- b: cell is nullable; it'd be nice to have something not-nullable that uses option
- p: on the other hand, separating nullability from ... borrowing,... option of slot or something. whereas you could just have cell. but... there are two separate concerns.  It seems more principled to keep them separate. otoh, people are like "what? a cell and a slot?" ... waffling.
- d: just because 2 things are different concepts, doesn't mean they have to be different things at the language level.
- p: we're talking about libraries already
- f: do we need to commit to 1?
- sully: we want to provide libs that people can use and not cry
- p: cell probably needs to grow dynamic borrowing anyway. we can add it to cell and see if it's distasteful. 
- bb: I'm pretty sure cell already has dynamic borrowing.
- p: kind of feels like ML ref cells.
- f: but... can we get mail on this?
- p: okay. actually, it does make it *quite* similar to ML. mutability only through cells or through mutable local vars.

## Iterators

- p: strcat started adding iterators, they went in, we found they were kind of awesome, our 'for' protocol is probably worse than external iterators, for various reasons
- d: external vs internal
- p: internal: uses a closure, like ruby, ML. like where you have ... like 'iter' in
- d: ocaml has loops.... not used much (except by graydon in the bootstrap compiler).
- p: basically it's a HOF.
- d: internal, because it's internal closing over its iteration state?
- p: internal not a good name, but external is. an external iterator is one with a .next or .hasNext method, where the... cursor vs callback-style is probably a better name for it. 
- p: strcat and I have done a survey of different languages, and most of them use external iterators in some fashion.  Ruby and ML are the exceptions. Haskell uses iteratees, don't know how they work
- d: ... have been improved on. Clojure has reducers, iteratee with better naming. Racket also use reducers (pumps, tanks, pipes). worth looking at
- p: haven't seen those. D: ... they all start with hasNext & next. proposal is to replace 'for' with something that calls 'next' repeatedly.
- d: does this produce a result? this is a subtle feature.  imperative vs functional is likely to cause big differences, tradeoff between functionality and performance. rust likes dependable perf model, but in the last 10 years, there have been advances in API design (not so much language design) worth looking at. really look at Racket and Clojure.
- p: rust has... so basically you call ...
code:
(chaining transformers together)
- d: even fps are doing it...

```
for x.iter().transform(|x| x+2).filter(|x| x%3 ==0)  {
     ...
}
```

- d: let's not just take the first thing that looks good. it's important that rust continues to repect historical precedent.
- p: strcat has looked at a bunch of languages, but I don't know about clojure, racket, etc.
- d: tell strcat to take a look at clojure & racket, I'll send blog post.
- jack: two good talks from rich hickey

http://clojure.com/blog/2012/05/08/reducers-a-library-and-model-for-collection-processing.html  
http://clojure.com/blog/2012/05/15/anatomy-of-reducer.html  
http://www.infoq.com/presentations/Clojure-Reducers  

- d: good if strcat could write up rationale, what we liked & didn't like about the various points in the design space.
- p: going back to why `for` is better this way: it makes break, continue, and return consistent. 
- cmr: Benefit of external: borrow checker works, no  way to guarantee break/loop/return etc works correctly with the current iterator protocol, no weirdness  about moving across  closure boundaries
- p: there's this performance problem. These also compile faster, because fewer closures.
- sully: llvm has a lot of work to dio in either case, may be too early to say how it'll run.
- bb: inlining vs devirtualizing?
- p: it's inlining of a non-constant function.
- sully: doesn't seem that hard, has to inline twice & propagate.
- p: sometimes llvm can't convince itself that the closure is remaining constant, that it didn't change during the loop, so it couldn't devirtualize it.
- p: but codegen args are not the strongest, as sully points out.
- p: we can't make a decision here without niko & graydon.
- bb: I favor getting rid of the funny for loop protocol. 
- f: so... there's a new funny `for`?
- p: no, it's not funny. ...
- d: I think it's really important that a language have a core looping construct, but that design space is non-trivial.

- d: question: what's the state of thinking about librarification of GC?
- p: stalled on discussing whether @ should remain as syntactic sugar.





















