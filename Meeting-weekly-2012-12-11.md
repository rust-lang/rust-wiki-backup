Agenda:

* directory modules
* kind names
* buildbot transition
* marking C functions unsafe
* AddAssign, etc.
* default methods behind a flag
* empty module scope
* INHTWAMA
* self -> Self
* Ord and Eq reform
* #[cfg()] improvements
* unsafe pointer indexing

# Directory Modules

- brson: I made the change to remove the crate language but now rust knows nothing about directories.  All of our creates have a bunch of patch attributes.
- brson: I was thinking about doing issue 4117 to get rid of some of those.
- graydon: I commented in the bug and am in favor
- pcwalton: +1

# Kind Names

- brson: I've always had this thing about the Send kind.  I want to push to rename Send to Owned and Owned to anything else.
- graydon: Completely agnostic.
- pcwalton: I kind of prefer send.  Just because I know some people in IRC have expressed confusion about the word Owned.
- pcwalton: OTOH, understand the concept of ownership is kind of central to Rust, so maybe it's good.
- brson: was this person talking about the existing Owned trait, or ownership in general?
- pcwalton: Maybe the Owned trait?
- brson: Because the Owned trait means no &, which is confusing
- nmatsakis: The name made more sense when we called managed "shared ownership"
- nmatsakis: in any case I want to replace Owned with the idea of having lifetime bounds on type parameters
- pcwalton: Wait, what? Has there been anything written about that?
- nmatsakis: I wrote an email at some point, I'll find it and summarize it.

# buildbot

- graydon: buildbot is now fully operational, should have enough machines to be online
- graydon: its snapshots are going to be the default snapshots that are downloaded
- graydon: there are two S3 buckets.  The old one is "dl" (for downloads) and the new one is "static.rust-lang.org".  Equally non-suggestive.  The new one contains the entire contents of the old one.  They will start to diverge over time.
- graydon: from here on in, when you register a snapshot, use the info from buildbot logs.
- nmatsakis: do we plan to decomission rustbot?
- graydon: sometime in the next month or so
- brson: how do you control buildbot?
- brson: how do I ask for a build?
- graydon: You go to one of the builders.  There are links at the top of the page.
- graydon: A builder in build bot is a queue of work.  A set of tasks scheduled to be done a certain way.  
- ... some discussion of how buildbot works that does not seem suitable for minutes  ...

# Marking C functions unsafe

- pcwalton: A breaking change so I want to do it early
- pcwalton: There was some disagreement on the mailing list
- pcwalton: Are we all ok with that?
- pcwalton: We could have a safe attribute but most everyone wraps functions in Rust wrappers anyhow, so I wouldn't expect this to have a large effect
- brson: I still agree.  The attribute seems like a weird way to make it safe.
- pcwalton: you could probably get C functions as unsafe today by writing unsafe as part of the sig.  It's kind of the wrong default.
- brson: to me wrappers don't seem like a big deal.
- pcwalton: for pow and stuff? won't that be annoying?
- brson: only one person ever wraps pow, and that's us
- pcwalton: it's not just pow, there are other C apis that don't take pointers
- graydon: it's not a lot
- graydon: how about this: a native function is unsafe iff it takes/returns raw pointers
- graydon: you can also declare it unsafe
- brson: I'm wary of trying to be so smart
- pcwalton: we've tended to get burned by smart defaults
- graydon: ok, it's really a question of deciding when things should be safe
- pcwalton: I feel like maybe it's better to err on the side of simplicity
- pcwalton: are people ok with the attribute?
- brson: it is going to change the type? I'm not sure if I'm ok with that, it'd be the only case where annotations affect the type signature

# AddAssign

- pcwalton: there are cases in which you want to overload `+=` separately from `+`
- pcwalton: it seems to me that perhaps the best way to do this is with a separate trait
- pcwalton: for example, strings with `+=` are inefficient
- pcwalton: hashtables is another example, if you use the IndexAssign trait
- pcwalton: C++ sort of manages to work around it but with a terrible cost, since they must create a hole if the element doesn't exist, leading to weird behavior if you fetch a key that doesn't exist
- pcwalton: therefore I'd rather not have desugaring
- nmatsakis: +1
- brson: so if you don't overload this trait, do you get the compound operators?
- pcwalton: no
- brson: it seems like a use for default methods
- graydon: it feels like you ought to be able to get nice default behavior
- nmatsakis: also there are types for which += makes sense but + does not
- nmatsakis: maybe the right setup is AddAssign as a base trait and Add extending it?
- pcwalton: that doesn't seem to make sense for IndexAssign
- nmatsakis: yeah, maybe it's not quite right, or not universally applicable

# Default methods behind a flag

- pcwalton: they don't entirely work right now, I don't know how critical they are for 1.0
- pcwalton: I would like to propose making an "experimental feature" flag
- pcwalton: I tend to feel that the half-implemented stuff should be more hidden
- graydon: I don't mind the idea of flags for experimental features
- graydon: I am surprised at the notion that default methods would not be considered critical for 1.0.  I agree with not critical for 0.5 but it seems to me that they an important part of the trait system.
- nmatsakis: I'm inclined to agree
- brson: there's a pattern with generic impls that fulfills many of the same use cases
- brson: impl<A: Minimal> A: Extended { ... }
- brson: default methods only give you the ability to *override* the default implementation
- nmatsakis: I think overriding is more useful 
- graydon: the distinction is that defaults can be overridden?
- brson: and the generic version requires importing the Extended trait too
- brson: so default methods may be more usable
- graydon: I still feel like default methods are a pretty strong component of the trait system
- graydon: I'm curious, when you say don't work right now, what's missing?
- graydon: legwork or semantic problems?
- pcwalton: a lot of legwork. cross-crate, objects, etc.
- pcwalton: static methods with defaults.
- graydon: anyway it's basically a matter of a lot of energy that hasn't been spent
- pcwalton: It's just a question of priorities
- pcwalton: I haven't personally felt the need for them very much
- brson: I feel like we should stop people from using them because they are so broken
- graydon: I'm fine with stopping people from using them for now
- brson: how will this feature enabling mechanism work?
- graydon: a -Z flag?
- pcwalton: I was thinking a crate attribute
- graydon: why not just a lint flag with the default being deny?

# Empty module scope

- pcwalton: It seems that we probably want modules to have a relatively empty scope?
- pcwalton: maybe there isn't a concrete enough proposal yet
- nmatsakis: the last time we had this conversation we... didn't conclude anything?
- pcwalton: right, we had some mention of a `..` operator?
- graydon: we could use `super` for it! same as with traits. `super::bar` vs `super.foo()`
- pcwalton: so your imports would be crate relative unless you write 'use super::foo'
- pcwalton: seems nice and tidy
- nmatsakis: it's fine with me.  as I recall this setup didn't solve the fundamental difficulties.
- pcwalton: I'm more concerned about ergonomics. I want to know where my names come from.  This is somewhat orthogonal from the implementation challenges.
- brson: so if I have a crate and I've "extern mod std" at the crate level, how do I access it?
- pcwalton: `use std::foo` will work just as today
- brson: what about if I don't use a `use` statement?
- pcwalton: no, resolving a path `std::foo` in an identifier should work the same way?
- pcwalton: well I guess what it means is "first look at my current module and if not go to the crate level".  
- brson: so are things at the crate level always in scope?
- pcwalton: depends on what you mean, modules yes but other items/types no.
- nmatsakis: why not? just because you don't want it that way?
- pcwalton: right, or else we could require the leading `::`
- graydon: that doesn't seem so bad to me
- graydon: what about importing from your children?
- nmatsakis: `use self::`?
- pcwalton: not a keyword right now?
- brson: but it probably should be
- pcwalton: I 
- graydon: I don't personally feel this one. Show of hands?
- brson pcwalton *raise hands with enthusiasm*, nmatsakis tjc *wave hands noncommitally*
- pcwalton: as a datapoint, in ECMA it was changed from our current system to a cleaner initial scope because people liked it better
- brson: this will break one pattern I've used in the past, the metadata module has a kind of fence around it right now
- nmatsakis: it once took me like 2 hours to figure out what the heck was going on there
- brson: no great loss
- pcwalton: maybe we can replace with a lint pass
- Summary:
    - use self::
    - use super::
    - use absolute::path
    - a path `p` is implicitly `self::p` unless you write `::p`
- graydon: you can currently do a use in a function scope, does that matter?

# 0.5

- pcwalton: I'd like to release 0.5 before the end of the year
- brson: we've been discussing this week
- graydon: apparently the world ends on the 21st
- brson: I think it might be beginning of next week?
- nmatsakis: I always had end of year in my head
- graydon: triage is needed
- pcwalton: docs need to be updated, that's the biggest thing
- graydon: can I perhaps ask a different question?
- graydon: at the beginning of this release cycle we planned and made a shortlist of things we wanted to get done
- graydon: the 2012-10-09 meeting
- graydon: 
