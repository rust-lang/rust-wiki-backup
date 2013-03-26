# Agenda

- Grammar
- 0.6
- "extern mod", "use", items ordering
- ~[0, ..n]
- "as @Trait" with coercion
- "@mut Trait"

# Attending

pcwalton, nmatsakis, brson, graydon, jclements, jld, metajack, tjc, azita, pnkfelix

# Grammar

- jc: Some of the lexer rules are a bit hairy.
- jc: It should parse all rust sources into token trees
- nmatsakis: Did you follow the discussion we had about how to deal with `>>` vs `>`?
- jc: proposal I recall was to actually make `>>` two separate tokens...?
- nmatsakis: not quite... let's discuss on IRC

# 0.6

- G: Still 70 on the milestone, too many but we've made progress
- G: We also closed about 50, not just bumped milestones
- G: I'm just doing robot babysitting, polishing release notes, etc
- G: I've got a list of bugs that I didn't close and I didn't demilestone because I wanted to discuss some of them
- G: Not sure if everyone else did that?
- Yes.  Everyone left 5 or 6 open.
- G: Release notes still coming into shape.  If I post a release candidate this evening...
- G: ...does anyone have anything queued up that may destabilize things?
- P: I've got a lot of big "break the world" changes that haven't landed yet
- P: I'm going to need a quick turnaround for a review in some cases
- G: So we won't do it today.  It's a short week (Friday off) in Canada.
- N: Shall we just take it day by day?
- G: I guess, Brian at least has the signing keys, so we could theoretically do the release on Friday.
- N: It's been suggested in the past that Friday is not the best day for a release anyhow, if the main purpose is to get people to take a look.
- G: Shall we review the remaining bugs?
- P: I have a few things to add to the agenda

# Extern mod, use items ordering

- P: Currently you can write `use` declarations and items in any order, but resolve always treats all items as shadowing use statements, no matter where they appear.
- P: This can be confusing and seems wrong.
- P: Also, `extern mod` shadows use statements but most people put it before the `use`
- P: My proposal is to require that you must write (1) extern modules; (2) uses; and finally (3) items, and resolve treats them as shadowing in that order.
- P: So there are two changes here:
    - flipping the shadowing rules for `extern mod` and `use`
        (e.g., `extern mod sdl; use sdl::sdl;`)
    - forcing people to write things in the shadowing order so as to give a better intution
- ?: Is it an error to have two top-level bindings with the same name?
- P: Depends, two functions yes, but use is not an error
- jc: What about multiple extern mods?
- N: My first thought was to ask why we permit shadowing at all?  But I guess that the sdl example is good.  Also `use`
- N: OK, if we permit shadowing, I think your proposal sounds good

# `~[0, ..n]`

- P: What should this mean?
- N: I think it should ideally mean `~[0, ..n]`, since we don't generally make the literal forms "compose" with ~ ...
- jld: yes, all the other cases of `~[` are actual `~` vectors
- G: You can get around it by parenthesizing, right?
- P: Yes.
- N: In the future it'd be useful to special case `~[None, ..n]` where the option is non-copyable
- P: I don't know the right thing to do there but we can defer that
- N: Yes, this just opens the door for that was my point

# `as @Trait` with coercion

- P: Niko has brought up that it might be useful to allow `@T` to coerce to `@Trait` automatically
- P: My question is (a) do we want to do this and (b) should we still allow an explicit cast?
- P: Keep in mind that right now we have three notions of "type comparison"---
    - Subtyping
    - Implicit coercion (`let x: T = ...`, perhaps `expr: T`)
    - Explicit casts (`as`)
- P: So I think there's a deeper question of "what does `as` mean"?
- P: I lean towards making `as` just for number conversion, 
- N: If we said that `as` is only for integers, it only needs to take a path, not a type, right?
- G: You could almost rip it out and replace it with functions...
- P: ...except for consts.
- G: I think casting to a boxed vtable representation is counterintuitive
- G: It's cute that we figured out that we can do it, but nobody will ever guess it
- G: So I'm fine removing that but I'm not sure what we ought to use to produce objects
- G: Magically coercing thing seems a bit off?
- P: Go does it and it doesn't seem too confusing
- N: It seems convenient to me, e.g., `do_printable(foo)` "auto-objects" `foo`
- G: So it's kind of analogous to how you'd use a trait parameter bound.
- N: That was the idea
- G: Do we have to decide now?
- P: No, we don't have to, 0.6 will not be totally backwards compatible
- B: My concern is that since objects are not that useful yet I don't know how imp't this will be
- P: I know that the Ruby users in the community complain :)
- P: A lot of people write little games with objects representing things in the game world
- P: And it's annoying to have to write `as @GameObject` all the time
- jc: I went through that, until I read the tutorial more carefully
- P: To me it feels like, we have had complains and Go uses it so there is precedent
- G: I agree with Brian that we don't have a lot of experience, but I don't see this change making things worse or causing us to be committed to a strong reliance on objects
- G: If anything it makes them a bit more invisible
- P: I do use `&` objects in sprocket-nes
- G: We use them in a few places, and we will use them more (e.g., `I/O`)
- N: They have a role.
- G: Like in C++, most everything is static, but not everything
- P: The invisibility with respect to I/O is a good point, because I'd like to have I/O yield the real types and not always objects
- P: But then people can write routines that just take `io::Reader` and things will still work
    fn read_my_thing(x: &Reader, ...)
- G: I'm ok with this change, does anyone object?
- G: I don't think this forces you to use objects more (or even suggests you should) but makes it a little smoother if you do use them
- jc: At one point Niko proposed adding `:` for a type assertion, is that related?
- jc: I always saw `as` as type assertion...
- pcwalton: ...`as` isn't good for this role because it does explicit coercion too
- jc: But is the `:` going to happen?
- P: that seems like a separate, backwards compatible discussion
- P: there is a discussion to be had about what *precisely* that should mean anyhow
- pnkfelix: actual proposal here is to restrict `as` to work with numbers?
- P: I don't have a strong opinion about it
- N: I think I like the idea of restricting `as` to numeric
- G: Is it possible for us to generate an `&Trait` in a constant?
- P: I don't know, but I think it's sort of tangential, coercions can happen there too
- N: I imagine we could make it work but I agree with P that it can be made to work
- G: I'm ok restricting it, I think if we remove it, newcomers will never even notice
# `@mut Trait`
- P: I think there is almost nothing to discuss, but...this should work, right?
- N: Not just `@mut` but `&mut` too
- P: Right, right
- N: Did you plan to put this in for 0.6?
- P: No...
- N: Ok.

# Bug review

- G: One bug I was concerned about was that `core::ops` is not showing up in the docs...?
- B: I fixed that (#4800)
- G: Cleanup: `rustc` vs `rustc.exe` on Windows?
- G: Feeding LLVM patches upstream? (#4259) 
- G: Does anyone even remember what they are anymore?
- B: I'm kind of working on that, Samsung has a number of changes that they are working on, and I've been encouraging them to bundle those up together to avoid disruption
- B: I've got a fork I gave to Samsung that removes all precise GC stuff
- B: There was one bugfix but I think it's upstream
- B: So I think there's not much else
- P: that might change with morestack?
- G: In terms of this release, is there anything where we feel like we've drifted so far it's scary...sounds like no.
- B: If we don't upgrade LLVM before the release, android will still be basically broken
- G: I think the premise that android will work for 0.6 is false
- G: In the 0.x series, our goal is just to throw something up for those who don't follow on a daily basis and show progress.  I mostly expect people to just see the tarball go by and not to really use it.  Let people know we are progressing, give release notes to read.
- G: I don't mean to minimize how much work went into it, it's a lot, but I think anyone who cares about the android port will have to use incoming anyhow
- Azita: So at what point will we be backwards compatible?
- P: Maybe 0.7?
- G: We don't know yet, I think it's hard to know what backwards compatible precisely means
- G: When we have a clear notion of how to define "BC" ... probably requires
    - bug queue under control
    - language versioning
    - some semantic cleanup
- P: To me, BC is a goal to work towards, not yet a promise
- N: I think there are degrees of BC.  So 1.0 is the true answer, but we've gotten almost all the syntactic changes out of the way thanks to pcwalton and others, I expect by 0.7 we'll be nearly 100% syntactically BC, but there will still be (hopefully small) semantic changes
- G: Biggest was probably 0.4 and we're still dealing with fallout, we haven't done change like that since
- G: I'm reasonably confident that we are converging now
- G: No new keywords added, only removed
- G: No new sections added, only removed
- G: Libraries were being moved around, cleaned up
- G: We are converging, but not there yet
- P: I'm pretty sure that 0.6 will be mostly BC, though "battle plans never survive contact with the enemy".  Some known things: copy keyword, as @Object, some borrowed pointer changes.
