# Agenda 01/21/2014

* default type parameters (acrichto) https://github.com/mozilla/rust/pull/11217
* lang items for primitive impls (acrichto) https://github.com/mozilla/rust/pull/11409
* fmt::Default => fmt::Format (acrichto) https://github.com/mozilla/rust/pull/11298
* no_mangle for extern fns (acrichto) https://github.com/mozilla/rust/issues/11089
* crate_type = "lib" (acrichto) https://github.com/mozilla/rust/issues/11253
* foo_opt => foo (acrichto) https://github.com/mozilla/rust/pull/11129
* What does \xNN mean? (brson) https://github.com/mozilla/rust/issues/2800
* env parameters redux (nmatsakis) 
* land BDW integration under a configure flag? (pnkfelix)

# Status

* acrichto: mutexes, queue backlog, small 1.0 issues, runtime guide
* brson: servo work week, android, roadmap, docs
* pnkfelix: Boehm GC integration, Finalization API experiments
* pcwalton: servo, mutexarc fixes, Scheduler<T>

# Friend of the Tree

* brson: Just one thing to do.  This weeks' FotT: steven fackler (sfackler).  Been contributing since last may; yet another CMU grad!  Increble amount of library improvements: base64, bitvector, IO modules.  RefCell.  io::util module, and most recently, external macro (syntax extension) loading.
everyone: *clapping*

# POPL update

* brson: Niko's tutorial went really well.  Some amazing slides.  Room basically filled on both days.  Someone was video-taping it too; brson asked the video taping person to send him a copy of the tape.

# lang items for primitive impls

* acrichto: documentation for primitives via a lang item for the primitive.  Technique for documenting primitives.  May extend to vectors/strings; not sure.  (And also not clear with advent of DST whether it needs to.)
* brson: why do you need a typedef?
* acrichto: gives you a spot to hang the documentation
* brson: why are vectors are problem?
* acrichto: because vector types have type parameters, and those type parameters can have bounds in different impls, so its not clear how to present that in the organization provided by rustdoc
* brson: think I like what you have for the primitives.
* pcwalton: I like it
* ... (further discussion felix missed)
* brson: land what you got

# env ptrs

* brson: At last mtg, we agreed we would move env ptrs to end of list
* brson: since then, eddyb has made case for removing it entirely
* brson: The reason: once closures are just a trait, then the env ptr will *be* the self pointer of that trait
* pcwalton: that makes sense to me
* acrichto: one shim per crate, or one shim per call-site?
* brson: answer is: there's no shim.  (or rather, one shim per function, because...:)
* brson: oh this patch also makes it so you cannot coerce fcn ptrs to closures; you can coerce named functions to closures, but not fcn ptrs.
* pcwalton: we should do that soon, find out what breaks
* acrichto: that changes language semantics.  worrisome.
* pcwalton: yes.  we should land it soon because we need to now what breaks
* acrichto: what about one shim per crate?
* ... (some discussion felix missed) ...
* acrictho: i will discuss further with eddyb

# boehm

* pnkfelix: built boehm, doesn't pass tests yet. still uses ref counting. may be useful to add this as a mode for compiling the runtime. should i put in the effort?
* acrichto: overrides unique allocations as well?
* pnkfelix: currently yes. might not need to override all unique allocations. e.g. c strings could keep using malloc free.
* acrichto: unfortunate that std requires all allocations to be through boehm
* pcwalton: agree
* acrichto: expect that if you link to libgc then allocators go through boehm
* pcwalton: isn't this what managed-unique is for?
* pnkfelix: that may be insufficient, may need to trace through borrowed pointers. don't know yet if it lets us hook boehm into the entire std lib
* acrichto: lang items for allocations redirect through boehm?
* pnkfelix: yes
* pcwalton: should be experimental
* brson: boehm may be sufficent for our gc for awhile?
* pnkfelix: don't know yet

http://www.hpl.hp.com/personal/Hans_Boehm/gc/license.txt

# fmt::Default -> fmt::Format

* acrichto: Motivation is need a deriving mode: #[deriving(Format)]
* acrichto: Don't really like this change since this adds stuttering, not sure we need deriving
* pcwalton: Weird that errors complain about 'Default' instead of 'Format'.
* acrichto: Also could use 'Display', 'Show'
* brson: not much better than the reflection-based formatting
* pnkfelix: recursively calls Default, so its different
* brson: I think we should have deriving, what to call it? Show?
* pnkfelix: alex?
* acrichto: It's ok.

# foo_opt => foo

* acrichto: Didn't get through this discussion. I'm reluctant to take it
* pnkfelix: Philosophy is to not bifurcate APIs. With unwrap noise I think the problem is that the failing one is the easiest to type.
* brson: If the goal is to make the failing version harder to type then doesn't making you type '.unwrap()' solve that?
* pcwalton: would rather have `.get()`.
* nrc: could maybe be solved with some syntax sugar. having to match Some/None is painful, unwrap is less painful, introduces failure, so want something even shorter. Root problem is matching is painful, so make that easier.
* nrc: any syntax is going to need to  be backwards compatible, so not necessary for 1.0


# \xNN escapes in strings
* brson: in some languages, denotes what we have (a code point).  in other languages, it always denotes a byte (i.e. code unit)
* brson: some people prefer one semantics or another
* brson: proposal: We officially leave things as they are.  Its the semantics of Scheme/Python.
* brson: Main downside: won't be able to copy/paste C string literals in all cases between C and Rust
* brson: any attempt to copy-and-paste a C string literal making use of this will be detected by the lexer (due to the properties of utf-8).  So there's very little danger of this biting people.

# no_mangle for extern fns
* acrichto: why isn't no_mangle the default for public extern functions?
* pcwalton: because it breaks the module system
* pcwalton: if libraries A and B both define extern functions `foo`, if you make no_mangle automatic/implcit, then library C cannot link againt A and B simultaneously
* brson: keep it explicit
* acrichto: okay.

# crate_type="lib"
* acrichto: crate_type="lib".  There are a number of different crate types one might select.  rlibs and dylibs.  But people will want to just write 'type="lib"' and get a reasonable default.  But what should that default be?  Or should it emit both types of libraries?
* brson: What is your opinion of what we should do?
* acrichto: I like one default: A compiler-recommend default.
* brson: I like having a platform-specific default.  (E.g. use-case of emcripten.)  Agree default should be static.
* brson: And what about a command-line override?
* acrichto: Separate issue.  If crate says it wants dylib and rlib, but command line says "i just want rlib", that's not going to work.
* brson: becaue our command line is additive?
* acrichto: because it stacks
* acrichto: we could have the command line specify the output format, while the crate specifies the *flavor* of that output format.  But I like having the crate specify its output format.

