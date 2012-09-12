# Agenda

- Minor syntactic cleanup (nmatsakis)
    - Purge `<-`, `<->`
    - Purge fn() expressions?
    - Require that `foo` in `(foo as @T) ` be an @ value
    - Rename DVec to GrowVec?
- Monomorphic instance explosion :)
- Tweak impl syntax before 0.4?
- Remove 'unchecked'?
- (tjc) Second thoughts on https://github.com/mozilla/rust/issues/3413 (maybe not needed)

# Minor syntactic

- `<-` and `<->`?
    - each have other, more flexible equivalents, deprecate and remove.
    - OK.
- Drop the `@fn(x: T) { ... }` form in favor of `|x| {...}`?
    - graydon: I've found myself having trouble pulling out subexpressions into closures
    - OK
- Require foo in `foo as @T` be an @ value?
    - `@foo as @T`
    - Double check whether `@T` is doubly indirect
    - OK
- GrowVec?
    - Nobody loves it
    - `VecBuf`
    - `StrBuf`
    - What else do we need them for?
        - Anything else?
        - Freezable structures don't require it
    - Ask bblum about dlist

# Monomorphic Instance Explosion

- graydon: we are creating a lot of duplicate code
- graydon: has anyone worked on C++ compilers? (no)
- graydon: reason we do such much duplicate usually bottoms out in using the size of a type
- graydon: feels like we are reproducing a lot of code in order to get step size to be a constant
- nmatsakis: are you sure that's the case?  and we're not using glue or something like that?
- graydon: yes, often that is true, but ... in type_use, purpose is to compute a bit map that specifies how much a polymorphic fn depends on its type parameters:
    - bit one: if fn depends on the representation in any way (size, alignment, take/drop glue)
    - bit two: if fn depends on the type descriptor itself
- graydon: but those are almost exactly the same
- graydon: but lots of code only depends on the size/alignment and not the glue
- graydon: maybe the answer is just to refactor
- graydon: I am not sure if it properly recycles, and not sure
- tjc: have you tried measuring to figure out how big a problem this is?
- graydon: I'm trying to figure out how best to do that
- graydon: hard to measure the effect without just fixing the code

# Tweak impl syntax

- brson: We've discussed it a few times.  The impl syntax is not entirely satisfactory.
- brson: Change `impl foo: trait` to `impl trait for foo`
- dherman: before it was kind of overwhelming with all the keywords and options
- brson: but without named impls it's much simpler
- nmatsakis: I'm basically +1, I seem to misread the current syntax
- dherman: I like 'type: trait'
- graydon: it's precisely the same as the constraint list, I'm happy with it as it is
- brson: ok, I'm not convinced there's a consensus to make the change

# Remove unchecked

- brson: is there any disagreement to removing unchecked?
- graydon: there's this longer term background talk about a more general effect system
- graydon: we could also currently label blocks with `pure`
- nmatsakis: this gets into what precisely borrowck should or should not permit in unsafe blocks, and I don't know what the answer to that is
- graydon: also, our lint modes already cover this concept of turning on/off checks
- graydon: perhaps the block form of unsafe should just be a lint mode `#[allow(unsafe)]`
- nmatsakis: chattier, but clear
- brson: you could turn off safety at the crate level
- graydon: yes though you still have the pure/unsafe qualifiers on functions
- nmatsakis: I'm not opposed to #[allow(unsafe)], particularly as it allows us to scale up to mutliple kinds of unsafe, which can help to an
swer the question of "what should borrowck allow in an unsafe block?"
- graydon: I do want to go in this direction.  I'm imitating Ada's system.
- graydon: I want to build this in from the beginning so that people can subset the language and so forth
- nmatsakis: seems related to this effect inference/discussion
- dherman: depends on how sound you want to be
- brson: this will make control over unsafety rather coarse, unless we expand where you can put attributes
- graydon: yeah it'll be at the function granularity unless we make blocks annotatable
- brson: kind of the opposite of what I prefer
- brson: graydon, are you think this is a change we could accomplish for 0.4?
- graydon: no
- brson: what about removing unchecked?
- graydon: how about switching unchecked to pure?
- nmatsakis: wasn't the goal to consolidate unsafe/unchecked?
- brson: yes though the motivator *was* removing keywords
- graydon: it *is* unsafe, that's true, mid-term if you just want to consolidate the two that'd be ok
- graydon: another nice thing about using the lint modes is that you can write #[forbid(unsafe)], though we can do that in other ways

# Issue #3413: second thoughts by tjc

- tjc: I made this RFC to make return or tail exprs implicitly move
- tjc: but I went ahead and made the moves explicit in libcore/libstd and I felt like maybe it's not necessary
- brson: I feel good about being explicit until we decide it's too painful
- graydon: I don't have a firm opinion, this is an optimization in C++ (return value optimization)
- tjc: we can still have a lint more to warn where you could be moving but you are not
- graydon: I feel like when we turn that on we will want `<-` again
- nmatsakis: bear in mind rvalues are always moves.
- graydon: ah, true.  probably doesn't happen all *that* often
- nmatsakis: I agree with brson let's err on the side of explicit

# Documentation

- brson: I keep putting off updating docs because...it's not fun
- brson: but we have tons of useful bugs and the docs are fairly out of date
- brson: I haven't actually looked but I suspect they are bad
- brson: we need some effort on that, I will do some, but...
- graydon: I am happy to help
- nmatsakis: I would be happy to do but do not have time till after the release
- dherman: doc sprint, even a single day, can be really helpful
- nmatsakis: how do people feel about having multiple tutorials?
- brson: pcwalton has started breaking it up, though I don't think it's fully integrated
- nmatsakis: I'd like to bring in my region tutorial
- dherman: multiple tutorials are good, easier entry point, and also helps you to leave things out (I can always write another thing)
