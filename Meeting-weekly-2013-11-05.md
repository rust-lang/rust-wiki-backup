# Agenda 11/05/2013
* wildcard pattern syntax (brson) https://github.com/mozilla/rust/issues/5830
* octal literals (brson) https://github.com/mozilla/rust/pull/10243
* vec repr (brson) https://github.com/mozilla/rust/issues/8981
* fate of libextra (acrichto)
* rustpkg priorities and transfer of ownership (tjc)

    See https://etherpad.mozilla.org/Rustpkg-priorities for "important" bugs

* role of fixed_stack_segments and friends in a post-segmented-stack world (nmatsakis)
* empty enums --> affine, does anyone care? #10278 (nmatsakis)
* temporary lifetimes: random rules or inference? (nmatsakis)
* http://smallcultfollowing.com/babysteps/blog/2013/11/04/intermingled-parameter-lists/

# Status
- acrichto: landing #10179, various bugfixes, now rewriting all uv bindings
- nmatsakis: landing multiple lifetimes, fixing mut permissiveness, dtors/temp-lifetimes
- pnkfelix: hacking on DST
- pcwalton: function reform
- tjc: wrapping up final rustpkg issues, OPW
- brson: removing struct deref, making wildcard patterns consistent

## wildcard pattern syntax
- brson: make the syntax for wildcard patterns consistent. Matching multiple things at a time is not uniform. strcat has a plan for * for "multiple things I am going to ignore". bjz suggested .. which is also used in vectors. I'm thinking go with .. Any opposition? Or thoughts?
- acrichto: in:
``` 
let Foo(..) = ...
```
- nmatsakis: in vector patterns ..  is followed by a pattern. But in tuples and structs we probably shouldn't do that for now. Not necessary from a parsing point of view. Could even make it optional in vectors.
- brson: No strong feelings, so let's move on.

## fate of libextra
- acrichto: lots of PR s for cool data structures, but we turn them down because they should go in libextra, but we don't have that yet. It should be sharded, and not official-but-curated collection. Can we do that now to put the cool stuff in there? One blocker was rustpkg, which is now in good shape for it. Does anybody have thoughts?
- pcwalton: Agree. Part of dogfooding rustpkg. We've been using it in servo, now in rust too.
- brson: Think we should create a rust-lang org to hold all of our repos. There will be an explosion. Then we should triage every module in extra to std vs. remote repo vs. we have to have an entry with a seprate crate for some odd reason. Beyond mechanics, not sure. Lots of build system hacking. Should get started on this now. It would help us expand the ecosystem because there are a lot of people who want to contribute.
- acrichto: So we create a repo and make the contributor an owner with push/pull privs?
- brson: Legal issues. If it's official Mozilla code, they need signed contributor agreements, licensing sorted out, etc. One thing is what do we do with automation for these repos. Can't just let it sit, need to make sure it all still builds against master
- tjc: have rust-ci, right?
- brson: Only tells us when things break.
- tjc: Can tell you when it last succeeded.
- acrichto: No notification if something breaks. Big problem is how do we put stuff in this incubator? Putting code there in the first place by a contributor and keeping it up to date is a little bit fuzzy. Maybe just needs some thought. Come back next week after thinking about it some more.
- brson: You want to own it?
- acrichto: ok.

## rustpkg
- tjc: I'm out in two weeks and will volunteer, but can't own it. Will triage the remaining issues. Started an etherpad. Add things that you need me to get done in the last few days. Trying to help kmc port servo right now. rustpkg is fairly stable, but not so stable that it should be without an owner. In a pretty good state to hand over. Please follow up with me.
- brson: I would like to take it. But not set to commit. Let's talk later. What's the state of rustpkg support for native code?
- tjc: Not terribly well-tested. Have a basic one. Native libraries are done with rustpkg knowing nothing about them. Write a build script that specifies files that are declared inputs (e.g. text files, libraries). Then also write code that replaces the build step that calls out to make the file. Would be nice to have a better model as the DIY expectation seems poor for the long term, but that has not been designed.
- acrichto: Also not a lot of documentation. 
- tjc: OK, deferred to later.
 
## stacks stuff
 
- nmatsakis: getting rid of segmented stacks. But what do we do when we call C functions? I assumed we would just remove all the machiner around annotating C functions with requirements, just say it's unsafe, and if the C function overruns it, it happens. Try to use a guard page, but that's it. Acrichto was suggesting we could require minimum stack requirements. And then start with 4MB and usually have 2MB left over when we get to C calls. 
- acrichto: I dislike signal handlers. If we do have growable stacks (guard page kills you, but auto map in otherwise), then we have to use a signal handler. If the C functions don't do the work themselves, then if we have a growable stack, we need signal handler machinery.
- nmatsakis: Still can use morestack to check for overruns. Now, we call malloc and rely on the OS to be lazy. Why does that not work in guard page scenario?
- acrichto: Though we were going to manually lazily map it in.
- pcwalton: Because I don't think the lazy mapping happens on Mac or Windows.
- nmatsakis: Then when we called a C function we'd be forced to allocate everything, right? We could handle it by inserting a bit in the code of functions that call C functions. What should the user model be? Do we expect users to annotate functions?
- acrichto: If you call a C function, the compiler automatically requests more stack (e.g., 2MB), but you can also personally require more or less. By default, no annotations, but a C call implies at least 2MB, but you can configure it up or down.
- nmatsakis: Just gets away some of the appeal, as it got rid of the whole complexity in C FFI calls.
- pcwalton: I agree with niko. Wanted to get out of the business of managing stacks.
- brson: Also has a limitation that you can't request a small stack at all. And servo will probably want to minimize.
- nmatsakis: If all C functions ask for 2MB by default, you would have to annotate every single C call in servo to have the [small_stack] annotation and any one messes it up.
- acrichto: OK, once we have guard pages in place. Never hit it in rust, but in C if you hit it, we'll abort the process.
- brson: I'm fine with that in native code.
- acrichto: Do you think we could survive without guard pages yet in place?
- nmatsakis: I think so. Guard pages are also independent of the mapping.
- acrichto: OK, destroying fixed_stack_segment. 
- nmatsakis: Removing the machinery and then two issues for guard pages and lazy/explicit stack allocation. Maybe other people can even tackle that for us.

## octal literals
- brson: 10243 PR adds octal literals to rust. Opinions?
- tjc: Hesitant to add more things, but it's useful for writing UNIX permissions. And want hex floating point, and once you go there, might as well have octal. 
- pcwalton: Little cost.
- jack: Can't we have arbitrary-base literals and be done with it?
- acrichto: 8bXXX 
- tjc: Most peple want decimal, binary, octal
- brson: SHA
- tjc: OK, so base 2
- pcwalton: lose the 0x syntax
- nmatsakis: I love 0b, too
- jack: Can use Nr for binary, where r is for radix. 
- acrichto: Small use cases
- nmatsakis: And can use macros for this. But I don't really care. Have no opinion except that we should keep 0x.
- brson: dherman?
- dherman: We removed octal literals from JavaScript because people were more likely to have bugs. We used C-style, which was horrible. Wasted a ton of time on if 0o is evil? Can the "o" be "O"? Octal literals matter (in JS) for UNIX chmod.
- pcwalton: lunar lander simulation, as the emulation uses them.
- nmatsakis: agree with jack that if we do anything we should do them all
- dherman: We can do it from scratch(ish). Make all radices lower-case only, since the uppercase stuff is hard to distinguish.
- pnkfelix: Do we support uppercase for the radix indicator?
- brson: Hrm, good question... 
- nmatsakis: summoning rusti!
- pnkfelix: I think we already require lowercase.
- brson: Any strong opinions? On anything? I kind of hate to just add minor features due to lack of dissent.
- pcwalton: I care. I want octal numbers so we can get UNIX file permissions.
- dherman: What do you think...
- pnkfelix: 0o ? or 0r ?
- jack: "0o is an affront to humanity?"
- pcwalton: 0o for now
- dherman: 0o is what we're doing in JS.
- brson: Then let's move forward wiht it

## empty enums
- nmatsakis: Comment, "for arbitrary reasons we make empty enums non-copyable". I don't know why that is because the bounds are usually just the intersection of all variants. Does anyone care? 
- pcwalton: Less special rules is better. 
- nmatsakis: If you have enum void with no variants (which you can only construct by unsafe means), is that copyable?
- acrichto: Seems fine if it's whatever by default.
- pcwalton: More consistent if it's copyable. Otherwise, need a special rule in the language.
- brson: Agreed.

## vec representation
- brson: strcat's proposal 8981. Move all of the fields (length, capacity, etc.) out of the allocation. Any thoughts?
- pcwalton: strcat's arguments are persuasive.
- nmatsakis: Good idea. But, there are variations to discussion. What is the representation for slices? Currently, a pointer+length. Case to be made for representing both owned vectors and slices the same way. Then the binary representation is identical, so you can do something very close to subtyping.
```
fn slice_all<'a>(x: &'a [~T]) -> &'a [&'a [T]] {
    transmute(x)
}
```
- nmatsakis: Similarly could slice all strings in an array without copying. Seems appealing. Downside is that they need a different number of fields, need them to be the same size. Or at negative offsets. Slice 3 words seems reasonable. This is hand-wavy, but might be worth investigating.
- pcwalton: Torn, because nervous about an extra word. For perf, it's gotten us in trouble with the Drop flag in headers. But your argument about transmute and slice_all is appealing. Different from C++, which also sometimes gets us in trouble.
- pnkfelix: What thing that C++ does?
- pcwalton: 3 words (in nmatsakis' extension)
- nmatsakis: Don't really have slices
- pcwalton: C++ ranges are 2 words. Go uses a 3 word rep for slice, since capacity is there, but it's generally considered a mistake in Go. Should make that word useless because otherwise you get strange semantics where people push onto parts of the array you didn't hand them.
- acrichto: Sounds like the change for owned vectors to be 3 words is good. Just slices are uncertain. Changing owned vectors is fine, but then we should benchmark before slices?
- pnkfelix: Owned vectors could also be 2 words. Then leave capacity at a negative offset.
- pcwalton: Middle ground that isn't very nice because having the capacity  inline is probably a win. Compared to transmute & slice_all, those are much more important.
- nmatsakis: Does the proposal imply removing the managed vectors `@[T]`?
- pnkfelix: Orthogonal
- pcwalton: Need to revisit with GC changes anyway.
- nmatsakis: Probably going to want to do initially as part of the GC / DST work, and might make this patch easier if there aren't as many cases to handle. Probably doable separately. 
- pnkfelix: strcat's PR is ready to go, isn't it?
- nmatsakis: Yeah, I think so. Let's do the change to owned vectors. Will open another one for the change to slices.

## temporary lifetimes
- nmatsakis: This week. Two options: when you create temp values (address of an rvalue), what lifetime do they get? a) syntactically based on where they appear
```
match foo() { ref r => ... }
```
- nmatsakis: Has to go into a stack slot, but where does it go and how long does it live? Could have rules (e.g., that lives as long as the match statement). Then:
```
let (a, b) = match compute_something() { (ref a, ref b) => (a, b) };
...
foo(*a, *b);
```
- nmatsakis: that's illegal because you're taking a reference into what was returned by compute_something. b) rely on inference and have some maximums of not living beyond function bodies, etc. Trade-off is some amount of clarity vs. flexibility. Clarity is a little compromised by how many special cases there are. Any opinions?
- pnkfelix: I prefer clarity, but I like to name subexpressions explicitly.
- pcwalton: Agree; try to make it syntactic because it's obvious to programmers when things are destructed. We can always expand later.
- Blog post: http://smallcultfollowing.com/babysteps/blog/2012/09/15/rvalue-lifetimes/
- nmatsakis: Inclined to agree. Was in favor of inference, but settled on simple rules as nice. See what fails to compile and report back.
- acrichto: One use case is the expansion of the format macro. Lots of weird things to get around lifetimes of rvalues. Would be great if some of those went away.
- nmatsakis: There are lots of cases where it's nicer (as less code) when things are inferred. C++ also has an in-between. Something like, "rvalues will live as long as they get freed unless attached to an rvalue reference in which case they live as long as that other thing." Kind of an adhoc syntactic way to let people extend the lifetime of a value. Comes down to basically the innermost enclosing statement. I'll look at the format macro and see why there is so much trouble. The current rules are very restrictive. Modeled after trans, which results in very short lifetimes.
- brson: Sounds good.
- pcwalton: backwards compatible? 
- nmatsakis: too restrictive but also not flexible enough (e.g., destructors running at the wrong time). I've done 2/3 of this work. Haven't done the frontend rules yet. Statically derived rules moving to inference later should be back-compat (as long as the rules are simple).

## intermingled parameter lists
- nmatsakis: Been thinking about it. There's a blog post: should read it.
- brson: I read it. +1 and liked.

## Other business?
Resolved: Rust is awesome.
