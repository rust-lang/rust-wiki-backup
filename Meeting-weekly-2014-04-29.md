# Agenda 4/29/2014

* rev_iter() (acrichto) https://github.com/mozilla/rust/pull/13648 
* nullable pointers (acrichto) https://github.com/rust-lang/rfcs/pull/36 and https://github.com/rust-lang/rfcs/pull/41
* allocator trait (acrichto) https://github.com/rust-lang/rfcs/pull/39
* trait matching (acrichto) https://github.com/rust-lang/rfcs/pull/48
* libstd facade (acrichto) https://github.com/rust-lang/rfcs/pull/40
* unsized/type/Sized? keyword (nrc)
* smaller refcounts (brson) https://github.com/rust-lang/rfcs/pull/23
* ufcs rfc (nrc) https://github.com/rust-lang/rfcs/pull/4
* tracking module ownership (pnkfelix) https://botbot.me/mozilla/rust/msg/13771360/
* bitflags macro (aturon) https://github.com/mozilla/rust/pull/13072

# Status

* acrichto: finishing up the bots, I/O timeouts, ffunction-sections, close_{read,write}
* brson: administrative, vpc, docs, box
* aturon: revamping process creation interface, c flags, adding some atomic ops
* pnkfelix: control-flow graph, check-stage1

# Action Items

* nmatsakis: close https://github.com/rust-lang/rfcs/pull/36
* nmatsakis: close https://github.com/rust-lang/rfcs/pull/41
* aturon: work with bjz to finish up https://github.com/mozilla/rust/pull/13072
* brson: close https://github.com/rust-lang/rfcs/pull/23
* nmatsakis: update https://github.com/rust-lang/rfcs/pull/48

# Nullable pointers

Discussing https://github.com/rust-lang/rfcs/pull/36 and https://github.com/rust-lang/rfcs/pull/41

< nobody taking minutes >

executive summary of minute-less discussion: No one champions either of the two proposals.  No one really rejects them out-of-hand either.  (Some discussion of whether we would want to close them or not based on that state; i.e. should closing an RFC be interpreted as "we will not ever do this", or instead as "we are not ready to think about committing to doing this right now, especially given the content of this RFC as drafted.")

- jesse: it sounds like https://github.com/rust-lang/rfcs/pull/36 is only needed because *T is allowed to be null
- nmatsakis: would like *T to not be
- acrichto: that's a whole can of worms
- brson: I guess we should close and gently explain that we're not ready yet

< missed minutes >

- brson: conclusion?
- dherman: reject based on being insufficiently fleshed out and revisit in the future.
- brson: specific problems?
- nmatsakis: vadim's: interaction between statics not clear
- nmatsakis: cmr: we're hoping that once the dust settles with smart pointers we can solve this in a more general way. we're more likely to 

# Module ownership

- felix: discussion on IRC. somebody felt like they should have been included in discussion on PR.
- felix: this person was listed in files as "active mainter" or "author". we don't have a way for any piece of code to know who should be cc'd. we mostly just know informally. do we want a more formal structure for knowing who to look 
- brson: have a proposed mechanism?
- jesse: seems like it needs bot support, so anyone can "subscribe to pull requests on these files"
- brson: 


# Bitflags

- aturon: working on library issue blocked on this. need type safe (?); we have EnumSet module, but it has issues. there's a PR for `bitflags!` macro: invoke with enumeration and it gives you something you can use as flags.
- aturon: PR been sitting there for a few weeks
- aturon:
- pcwalton: I want this. Have one in servo.

- nmatsakis: do we plan to remove EnumSet?
- aturon: I'm not sure. In your commentary for EnumSet you said there were two use cases: 1 working with C (bitflags!); 2) for Rust code (EnumSet). I'm ok with having both.
- nmatsakis: does seems like they serve different roles. With bitflags you lose the ability to map.

# Smaller refcounts 

https://github.com/rust-lang/rfcs/pull/23

- brson: Same arguments here as last time about the wasted word.
- brson: I'll close

# rev_iter

- acrichto: https://github.com/mozilla/rust/pull/13648
- acrichto: looks good to me
- pcwalton: +1
- brson: let's do it

# libstd facade

- acrichto: https://github.com/rust-lang/rfcs/pull/40
- acrichto: takes std and shards it into smaller libs and makes std the interface
- acrichto: this would let us put collections into std, other things like that
- acrichto: haven't seen much opposition
- nmatsakis: I like it
- pcwalton: works for me. some comments talk about collections and allocators
- acrichto: our allocator story isn't fantastic. big assumption in libcollections is allocation can't fail. right now that allocator is libc; shouldn't assume libc allocator. not backwards incompatible
- pcwalton: fixed by making collections parametric over allocator

< missed >

- pcwalton: you can say the default allocator calls some functions called "rust_malloc" and "rust_free" but the actual definitions are swapped out.
- acrichto: how do people feel about the core library not having strings at all? Needs a bunch of methods that allocate.
- nmatsakis: prefer not having special rules for strings


# Trait matching 

https://github.com/rust-lang/rfcs/pull/48

- acrichto: This provides a solution for Eq/TotalEq
- nmatsakis: fundeps/associated types are controversial. thinking of removing them from RFC and doing a followup. would still solve your problem
- nmatsakis: I'll do a slimmer RFC.
- felix: how does this affect Eq/TotalEq?
- acrichto: dealing with total vs. non-total. it uses trait matching to enable...

issue link: https://github.com/mozilla/rust/issues/12517
https://gist.github.com/alexcrichton/10945968
