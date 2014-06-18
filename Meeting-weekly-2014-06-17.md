# Agenda 6/17/2014

* older RFCs (nrc) - https://etherpad.mozilla.org/RPT2iP80Iq
* remove f128 (acrichto)
* partial_cmp (acrichto) https://github.com/rust-lang/rfcs/pull/100
* libserialize rewrite (erickt) https://github.com/rust-lang/rfcs/pull/22
* putting #[unsafe_destructor] behind a feature gate (pcwalton)
* removal of cross-borrowing (pcwalton) https://github.com/rust-lang/rfcs/pull/112
* Index to three traits (pcwalton) https://github.com/rust-lang/rfcs/pull/111
* int inference (brson) https://github.com/rust-lang/rfcs/pull/115
* option/result API (bjz) https://github.com/rust-lang/rfcs/pull/113

# Attending

zwarich, cmr, brson, nrc, huon, azita, dherman, bjz, aturon, pcwalton, niko, pnkfelix, jbailey, luqman, acrichto

# Status

- brson: home stretch of automation overhaul, administrative
- nrc: DST - remove TraitStore
- felix: landing dataflow atop cfg; new dtor semantics analysis
- cmr: libsyntax
- pcwalton: unboxed closures, P-backcompat-lang burn down
- acrichto: cleaning up the facade, cargo work ahead, removing @, removing ~[T]
- nmatsakis: [#5527](https://github.com/rust-lang/rust/issues/5527)
- aturon: lib guidelines, task API, stability index, Command improvements, COW
- luqman: various coercions issues

# Action Items

- (brson) close https://github.com/rust-lang/rfcs/pull/80
- (brson) accept  https://github.com/rust-lang/rfcs/pull/89
- (acrichto) close https://github.com/rust-lang/rfcs/pull/67
- (niko) accept https://github.com/rust-lang/rfcs/pull/66
- (acrichto) accept https://github.com/rust-lang/rfcs/pull/100
- (aturon) work toward consensus on https://github.com/rust-lang/rfcs/pull/113/
- (brson) update https://github.com/rust-lang/rfcs/pull/115 and ping acrichto to merge
- (pcwalton) update https://github.com/rust-lang/rfcs/pull/112 and ping niko to merge
- (pcwalton) collect data on https://github.com/rust-lang/rfcs/pull/112

#  Old RFCs (https://etherpad.mozilla.org/RPT2iP80Iq)

## RFC PR 80 (Unsafe Fields, https://github.com/rust-lang/rfcs/pull/80 )

- nrc: What do we want to do here? Discussion seems to have stopped.
- brson: This has come up many times and we've always said no.
- acrichto: This looks a lot like Unsafe<T>
- huon: This isn't internal mutation. Modifying the field is unsafe, but you still can't mutate it whenever you want.
- pcwalton: I want this but I don't want this now.
- (others): Unsafe<T> exists, let's try just using that first.
- nrc: Does the compiler stop you from accessing an Unsafe<T> outside of unsafe code?
- nmatsakis: It's a private field only accessible via an unsafe function.
- acrichto: Not quite, it's a public field to be initialized in a static.
...
- acrichto: I think this should be postponed.
- nmatsakis: I guess that would be a different type. I guess this RFC addresses initializing in (... statics?)
- felix: The RFC itself is also incomplete.
- nrc: I think the only thing left in the RFC is deriving?
- felix: I don't think that's true, for example whether initializing the field should require unsafe...
- pcwalton: Agreed.
- brson: Can we punt this and move on?
- nrc: I agree it can be postponed post-1.0

## RFC PR 89 (Loadable Lints, https://github.com/rust-lang/rfcs/pull/89 )

- nrc: Conversation seems to have stopped there, but not definitively in either direction.
- nmatsakis: I thought we discussed this?
- brson: He had lints and plugins as separate RFCs, we discussed plugins.
- nmatsakis: This would all be feature gated
- acrichto: Not necessarily, just the plugin registrar etc.
- nmatsakis: That's what I mean, all the stuff I'm worried about.
- brson: Has anyone who has read the RFC have an opinion?
- acrichto: I read the implementation, it's unfortunate that the lint pass trait is separate from the Visitor trait. I'm uncomfortable with it.
- pcwalton: I really want loadable lints because I think it's needed to make Servo safe, and I'm automatically in favor of anything needed to make Servo safe.
- nrc: Is user-loadable lints something we want?
- brson, nmatsakis: yes

## RFC PR 67 (User-friendly input macros, https://github.com/rust-lang/rfcs/pull/67 )

- nrc: This is something I want, there's no implementation though.
- brson: I do think this is an area that we're clearly missing.
- pcwalton: I think we want it but we should put it in a library and follow the normal library inclusion process.
- brson: I agree, someone should figure this out out-of-tree. Are you proposing to implement this yourself?
- nrc: Yeah, I've put some work into this, but I don't want to do more if we don't want this.
- acrichto: It's very useful for writing little command line utilities. I think we should postpone this and do it in a library.

## RFC PR 66 (Better temporary lifetimes, https://github.com/rust-lang/rfcs/pull/66 )
- nrc: Niko said he was going to propose something possibly better.
- nmatsakis: I changed my mind. I would consider accepting this RFC, but it has some under-specified parts. The basic idea is that currently we have syntax-based rules to decide what the lifetime of a temporary is, and if we decide that it's being assigned to a local variable we give it the lifetime of the block, otherwise we used the innermost enclosing statement (?). These rules cause problems (`let y`, `foo`). He proposes rather than doing a purely-syntactic rule, after typechecking, we will look at whether the argument also uses the lifetime of the return value, we will extend it to the lifetime we would otherwise (?). I think it's a good idea to address this, both for ergonomics and [to make our rules less anti-abstraction] (`Foo { ... }` vs `Foo::new`).
```
let x = &3; // the temporary that holds the rvalue has the lifetime of the block
let x = map.borrow().insert(...); // the temporary holding result of borrow is just the current stmt
let y = foo(&3); // lifetime of temp holding 3 is the statement
let z = *foo(&3) + 3; // here lifetime would be innermost statement
fn foo<'a, T>(x: &'a T) -> &'a T { x }
let x = Foo { x: &3, y: &5 } // ok
let x = Foo::new(&3, &5) // error
```
- brson: Does this completely replace the issue around non-lexical borrows?
- nmatsakis: Completely orthogonal. This is only talking about making temporary values and how long those should live.
- brson: Do you want to accept this RFC as is or are there changes we need?
- nmatsakis: It could be more specific, but I don't know if the details should go into the RFC or not.
- felix: This affects RAII right? (...)
- nmatsakis: That is true. This still isn't full-on inference, the things you need to take into account are relatively simple.
- pcwalton: Presumably this is not backwards compatible?
- nmatsakis: Everything that it accepts is currently an error, so it's backwards compatible.
- pcwalton: It might make borrows last longer than you expect.
- nmatsakis: There might be some corner cases where a destructor runs later, but I think those will all be errors today.
- pcwalton: Let's assume it's backwards compatible.
...
- brson: Let's accept this.
- pnkfelix: Just in case we are worried about the potential complications to RAII, should we consider limiting the effects of this feature (namely the extension of object extent based on types and lifetimes), limiting those effects solely to objects that do not implement Drop
- nmatsakis: I think that sounds more confusing.
- pcwalton: I think we should reserve the right to change our destructor semantics post-1.0.  C++ is very squishy in this regard; e.g. there are cases in the standard that say that the implementation defines whether the destructor is run zero, one, or two times.

# RFC PR 100 (partial_cmp, https://github.com/rust-lang/rfcs/pull/100 )
- acrichto: Adds a `partial_cmp` method to the `PartialOrd` trait. It's easier to implement this one method than all of the other methods. The plans to the `Ord` trait would add all of the methods to the `Ord` trait, bringing the two traits in line. I believe the default methods were the impetus for this RFC. To me, `PartialOrd` is an implementation detail that almost no one is going to see. I view this as Ok.
- pcwalton, nmatsakis: Yeah
- brson: Let's take it.

# Feature gate `#[unsafe_destructor]`
- pcwalton: It's clear this isn't what we want, and there's been some malaise about this. We should feature gate this....
- (general agreement)
- acrichto: If we do this, we still need to fix some issues around this for 1.0 right?
- pcwalton: I don't think so.
- acrichto: There's some code ...
- nmatsakis: It's not possible to export a safe interface using RAII today. I want to fix this.
- pcwalton: Do we have bugs on this? I want to make sure every backwards compatibility issue is marked.
- acrichto: I just don't want this to say we won't fix destructors for 1.0
- pcwalton: I think 8142 is solved by this, 8861 not 
- brson: Let's do this.

# RFC PR 113 (Result API, https://github.com/rust-lang/rfcs/pull/113 )
- bjz: I want to standardize the APIs for Option and Result where they make sense, using an adaptor type for operations that are biased towards `Err`, seems to be ...
- acrichto: Looks like this also adds a new module to the standard library with the `Expect` trait.
- bjz: I'm not sure if that's necessary.
- brson: It seems there's a lot of dissent here about verbosity.
- aturon: Those have been addressed by changes to the RFC.
- aturon: I'm in favor of making the APIs consistent. I don't feel like there is consensus around this proposal yet, don't want to merge it yet.
- bjz: We don't need to make any decisions here, I just want to bring it to people's attention.
- brson: Let's get more feedback on this so we can make a decision.

# RFC PR 115 (Removing int inference, https://github.com/rust-lang/rfcs/pull/115 )
- brson: nmatsakis had already implemented, basically no disagreement. One open question about enum discriminants.
- acrichto: I believe that the signedness of discriminants does not matter anymore.
- nmatsakis: I think they need a type?
- pcwalton: They are "atyped" rather than untyped now, you cast it to get a specific type.
- nmatsakis: Should they be typed to some unresolved integral type?
- pcwalton: I believe we should just look at the rhs of `as`
- nmatsakis: Are you saying we shouldn't type enum discriminant expressions?
- pcwalton: Yes.
- nmatsakis: We could do this with an unresolved integral type.
- pcwalton: Sure.
- nmatsakis: That's a cute idea. It can be *some* integral expression, but you can't be specific about what it is. Or we could accept any integral type.
- pcwalton: I believe we should accept any type.
- cmr: Will we just truncate to a smaller integer type?
- pcwalton: Yes, it will be exactly as if you had typed whatever the enum discriminant value is and then casted it.
- brson: Ok, I'll modify the RFC and ping Alex to merge it.

# RFC PR 112 (Remove cross-borrowing https://github.com/rust-lang/rfcs/pull/112 )

- pcwalton: I think everyone wants this.
- nmatsakis: I want to remove all special casing of Box, does this handle that?
- pcwalton: I believe those should be separate RFCs.
- nrc: I would much prefer getting rid of coercions to &mut but keeping the ones to &.
- nmatsakis: I would like to add that in a more general way. I think the whole point of going to Box is that it isn't special. There's a certain amount of ambiguity in those coercions. I think we should address this (w?)holistically rather than keeping these special cases.
```
fn foo<T>(x: &T) { /* ambig. if you want to have auto-ref and cross-borrowing */ }
foo(box 3)
```
- nrc: I feel like this is going to make things massively more inconvenient...
...
- acrichto: I haven't collected data, but I imagine as_slice is way more common than &*
- pcwalton: I don't think it will be that bad, given that this doesn't apply to ...
- nmatsakis: I kind of agree with you Nick, but I want to gather data on what coercions we're doing, but I don't know if we can remove all of them because it won't be clear which ones you can apply.
...
- nmatsakis: one reason I am not keen on this particular `Box<T> -> &T` coercion is that it doesn't jive with how I think about boxes. I think of them as values, so it surprises me that a unboxed value requires a `&` but boxed values do not:
```
let x = Vec::new();
foo(&x)
let x = box Vec::new();
foo(x);
fn foo(x: &Vec) { ... }
```
- brson: I agree with Niko and Patrick, and in general we've been paring down the language. This is another instance of that, and I'm not against doing it one more time to get rid of weirdness.
- nrc: Are we going to want this for other smart pointers, like Rc<T>?
- pcwalton: I don't know. I think we should do the same thing with Box and Rc. I think that consistency is more important... I would like to be as consistent as we can for 1.0.
- brson: One think Niko brought up is that at the next workweek we can work specifically on ergonomics. We've spent all this time making the language consistent, let's spend some time making it usable again.
- nrc: I'm worried about getting a really crufty compiler.
- pcwalton: This is water under the bridge at this point. That is a concern, but it's already happened.
- dherman: If what we end up with to ship something by a deadline is a messy implementation, that's success. The least of my concerns is if the implementation is messy, but rather whether we are shipping something on time and what we wanted to ship. If we ship something and say "it's really beautiful semantically and later we'll clean it up to make it something you want to use." I'm being hyperbolic here, but this is an important marketing issue.
- pcwalton: How about we just remove the &mut for now, and leave in &
- brson: Let's do this.

# f128

- acrichto: Does anyone object to removing f128? If anyone says yes, I'll bring it up next week.
- huon: Yes!