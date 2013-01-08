## Attending

Graydon, Tim, Dave, John, Niko, Brian, Patrick, Azita

## Agenda

  * Region syntax (nmatsakis)
  * "write barriers on @" aka INHTPAMA aka INHTWAMA aka &alias mut (nmatsakis)
  * Planning meeting later on! (graydon)

## Region syntax

  * **Niko**: want to poll people's opinions
  * **Niko**: current system has some aspects I'm unsatisfied with; too many defaults & tries to be too smart
  * **Brian**: it's become the story of Rust
  * **Niko**: yeah
  * **Niko**: main thing that's too smart for its own good: lifetime parameterized type like struct or enum; right now we infer that it's lifetime-parametric
  * **Niko**: user writes: `field: &T` and we infer containing type is lifetime-parameterized, and use name `self` for that
  * **Niko**: what ends up happening is people don't have appreciation they're doing something mildly complicated; they write code trying to use this and it doesn't work
  * **Niko**: want to make things more explicit. imagine that having to write explicitly that type is lifetime-parameterized might help give them idea what's going on
  * **Brian**: to use this type you have to know something's going on, but to create the type doesn't require that knowledge
  * **Niko**: if you're gonna use the type you may or may not have to know something's going on, but you probably do. in practice seems to happen often enough
  * **Niko**: if you use type inside another region pointer, will select value for lifetime parameter based on context; if you have region pointer to `foo` where `foo` is lifetime-parameterized struct, lifetime of the region parameter will be the lifetime of contents
  * **Niko**: happens a lot in constructors. people tend to write a constructor that returns a `Foo` by value, people will take contents
  * **Brian**: they probably ask on IRC for the syntax
  * **Niko**: and the syntax isn't well-documented & I'm embarrassed by it
  * **Niko**: a little orthogonal
  * **Niko**: first: does everyone agree should be more explicit in places? second: if so, what syntax should we use?
  * **Niko**: for the latter I posted some alternatives on my blog
  * **Niko**: at least wanted to make sure people agree on first
  * **Brian**: agree
  * **Dave**: nods head
  * **Graydon**: agree
  * **Niko**: how far in explicit direction do we want to go? keep that on function declaration don't have to declare
  * **Niko**: function implicitly parameterizes right now; I tried it out and it was more verbose than I anticipated
  * **Niko**: only defaults would be if you write an anonymous lifetime like `&Foo` but everything else would be fully explicit
  * **Brian**: does that mean internal lifetimes have to be explicit?
  * **Niko**: meaning?
  * **Brian**: inside structure. since you can't name them in function signature
  * **Niko**: don't know. there's a lot of questions. related I guess
  * **Dave**: feels like you have to play with examples to turn the dial to "just right"
  * **Niko**: if we make things more explicit we gain more flexibility with syntax. we could use resolve to figure out if something is a type or a lifetime
  * **Brian**: maybe ask for example code on mailing list
  * **Niko**: we can resolve the exact syntax later, but sounds like there's agreement that there's an issue

## Write barriers

  * **Niko**: at some point I proposed change to language that would move what's statically guaranteed into dynamic checks, which I call write barriers
  * **Niko**: static rules are too strict in practice to really use effectively
  * **Niko**: you can write the dynamic checks by hand but becomes cluttered and hard to read
  * **Niko**: main change is to `@mut` type, which means a managed pointer to mutable data
  * **Niko**: right now that can be aliased so borrowck is very conservative
  * **Niko**: if you have `@mut int` you're probably fine, but if you have a complex type with structs or vectors, there's very little you can do with it besides read it. even if you read it, you have to be pure. extremely restrictive
  * **Niko**: instead, we'd make those types dynamically freezable
  * **Niko**: you could take a pointer, freeze it by creating an immutable alias to it. that would set some bits saying it's frozen
  * **Niko**: when you write to it, we check those bits and program will fail
  * **Niko**: can permit you to use these naturally so long as you don't modify an immutable pointer
  * **Brian**: connected to proposal for making `&mut` aliasable?
  * **Niko**: separate proposal.
  * **Patrick**: `&mut` more common. if we didn't make `&mut` borrowable to `&imm` we'd have to make another kind of pointer.
  * **Niko**: `~const` or something
  * **Patrick**: you'd have a data structure with mutable stuff in it, and an array you want to iterate, which would be impossible
  * **Brian**: if we don't add write barriers to `@` couldn't borrow at all from `@mut` under this unaliasable-borrowed-pointer scheme
  * **Niko**: right
  * **Niko**: it's like a read-write lock
  * **Brian**: it has to convert the checks to dynamic, right?
  * **Niko**: at the point at which you borrow it
  * **Brian**: so you can't do one of these without the other
  * **Niko**: that's right
  * **Niko**: I hadn't considered that combination, but you're right
  * **Niko**: I don't know whether that explanation was helpful or not
  * **Niko**: with this proposal, every time you have `&mut` we guarantee that's the only pointer to the data that's being mutated. that gives borrow checker more assumptions, and it turns out usually to be the case
  * **Niko**: example: if you're modifying a hashtable, and it took a pointer to itself, it would know there's no way for it to recursively modify itself
  * **Graydon**: essentially taking ownership when you borrow an immutable pointer
  * **Niko**: yeah; different kind of ownership. more exactly akin to borrowing with real life borrowing. real life borrowing instead of amazon.com borrowing.
  * **Graydon**: other kinds of borrowing are just read-only borrowing.
  * **Graydon**: didn't actually understand the discussion in the middle
  * **Dave**: neither did I
  * **Niko**: summary: borrow checker will give you fewer errors
  * **Dave**: downside: more machinery in runtime, feels like it goes counter to Rust goals of simple runtime model
  * **Patrick**: only occurs with @, which has always had stuff going on
  * **Dave**: good point
  * **Patrick**: at some point @ will need write barrier *anyway*
  * **Patrick**: my biggest concern is that this is like `ConcurrentModificationException`. failure you get if you try to borrow something mutable as immutable, or something immutable as mutable. those are both failures
  * **Brian**: also hard to figure out where failure is
  * **Dave**: whole-program, b/c it's dynamic
  * **Patrick**: but will always be on the stack
  * **Dave**: excellent point
  * **Patrick**: <background on `ConcurrentModificationException`>
  * **Patrick**: this is similar b/c this is usually where it comes up: you're iterating over an array which requires it be immutable (important for memory safety), creating a mutable pointer will result in an exception
  * **Niko**: only managed array
  * **Patrick**: yes
  * **Patrick**: might seem that this never happens in Java so we're probably fine
  * **Patrick**: not quite: checked at time you borrow, not at time you mutate. could be taking an inocuous looking `&mut` borrow and you get the concurrent modification exception
  * **Dave**: does that have to be the case?
  * **Niko**: otherwise would need many more write barriers
  * **Dave**: ah, right
  * **Niko**: I think it's the right thing to do. most of the time when you're modifying data you're the only one modifying the data. when you have a pointer to an object and writing code that manipulates it, you're not thinking about other code modifying. even in sequential code you get these kinds of "race conditions"
  * **Patrick**: this is totally unsafe in C++ in iterators
  * **Graydon**: a couple additional ways to think about this? a bit like a reader-writer lock -- pretty much is. can we have an api for asking if a box is currently being held?
  * **Niko**: no reason why not
  * **Graydon**: can we have a way to sleep on it?
  * **Patrick**: will always deadlock
  * **Graydon**: right, not concurrency in thread sense
  * **Graydon**: that is something you're gonna wanna be able to query
  * **Dave**: could have an option version of the API
  * **Graydon**: as long as it can be dynamically detected, not so much weirder than playing games with option. you wind up making a `Cell` type
  * **Patrick**: more like `Mut`. `Cell` is for one-shot closures
  * **Graydon**: rather than stubbing your toe on it for 6 hours and wandering into IRC, I agree let's build it into the language
  * **Brian**: will this eliminate need for `Mut`?
  * **Graydon**: yeah
  * **Niko**: yeah
  * **Patrick**: necessitates getting rid of mutable fields?
  * **Niko**: don't think absolutely necessitates, but makes more sense if you do it.
  * **Niko**: that's the third leg of the stool: I proposed getting rid of `mut` fields and moving mutability to owner
  * **Brian**: like it in principle, but it's going to affect semantics of ARC and other concurrency things
  * **Niko**: unsafe code is unsafe
  * **Brian**: ARC depends on properties of things that have mut fields right now
  * **Patrick**: to prevent cycles. OTOH I've argued cycles are too useful in ARCs to forbid. at least, forbidding ARCs in ARCs is too conservative
  * **Brian**: it's solvable anyway
  * **Niko**: thing that worries me: say you're writing traditional Java-like code, OOPish code. you've got objects being mutated. say `Player`, and you write `@Player` when you reference it. you couldn't make just the one field mutable. have to do `@mut Player`
  * **Patrick**: that's probably better. if you have `@mut` dangerous, can't give that up if it's one of your arguments
  * **Niko**: if you're doing callbacks, might want `@mut`
  * **Niko**: seems similar to: recent work on safer threading, read-only modifier. similar but inverse: when you plan to mutate things have to do `mut`. to some extent can be done with typedefs: `type Player = @mut struct Player`
  * **Patrick**: goes against our style guide
  * **Niko**: well, maybe we change it
  * **Niko**: I think it's good but that particular pattern will look different
  * **Patrick**: I think our community will like having it explicit
  * **Patrick**: can't hide mutability in your type, gotta say it
  * **Dave**: but can't say a particular field is immutable anymore
  * **Niko**: could add that back with `const` qualifier or something
  * **Dave**: <screws up face>
  * **Patrick**: can use privacy to get immutable fields
  * **Dave**: not obviously "more explicit"
  * **Patrick**: more explicit *at use site*
  * **Dave**: losing some finer-grained invariants
  * **Patrick**: less expressive, for sure
  * **Graydon**: back up: can you explain motivation?
  * **Niko**: currently inherited mutability overrides immutability
  * **Dave**: it eliminates the weirdness of inherited mutability
  * **Patrick**: "imagine never hearing the words aliasable / mutable again" -- if you have a unique array in a mutable field, you can never iterate over that array. that's obviously a footgun
  * **Graydon**: yeah
  * **Patrick**: second issue is what Dave brought up: having mutability on fields bifurcates types into freezable and non-freezable types. because mutability on fields is familiar for people used to languages like C++, they'll accidentally create types that are unfreezable
  * **Graydon**: I'm not so worried about people doing C++ style
  * **Patrick**: it's actually an Ocaml-ism
  * **Graydon**: yeah
  * **Niko**: there might be a way, don't know if it makes sense, to interpret mut qualifiers on fields in a way that's compatible, to get more like semantics Dave's talking about
  * **Patrick**: might be better to have a type as always mutable
  * **Niko**: declare a type as mutable or something?
  * **Patrick**: basically some notion of `~mut`, or a mutable cell
  * **Patrick**: not sure this is useful anyway
  * **Niko**: merits some thought, anyway
  * **Brian**: can we do rest without implementing that?
  * **Niko**: yeah
  * **Niko**: have to make sure semantics are sound around that. my goal with the post was to eliminate that error message from compiler b/c it just can't happen
  * **Niko**: one extension Patrick proposed: allow `&alias mut` and still get the error
  * **Brian**: might get through entire codebase and find we don't even need it
  * **Niko**: might be needed in other codebases; Patrick had DSP example
  * **Graydon**: don't understand this third leg of the problem. aliasable `&mut`...? that's a separate proposal?
  * **Niko**: I think we can separate it
  * **Niko**: it's basically if `&mut` semantics changes to be quasi-ownership, maybe you still sometimes want old semantics
  * **Niko**: might want to have multiple mutable borrowed pointers to same data
  * **Graydon**: suppose you don't do that. if you don't, are you still allowed to have immtuable pointers peeled off from it?
  * **Niko**: from what?
  * **Graydon**: I have an `&mut T` -- can I borrow an `&T`?
  * **Niko**: assuming latter implies quasi-ownership?
  * **Graydon**: yes
  * **Niko**: then yes
  * **Graydon**: immutable borrow masks the mutable borrow.
  * **Niko**: yes, just like borrowing a mutable local variable
  * **Graydon**: statically
  * **Niko**: yes
  * **Graydon**: not based on write barrier. that's only when I borrow from shared box
  * **Niko**: that's right
  * **Graydon**: so question you're wondering is whether makes sense to borrow another `&mut T` and have two of them pointing to the same thing at the same time
  * **Niko**: that's right. would probably have to be spelled differently
  * **Graydon**: what's wrong about it, right about it? why doesn't it work?
  * **Niko**: to allow you to reborrow as immutable, to make statically safe, can't have aliasing
  * **Graydon**: why not?
  * **Niko**: you reborrowed from one alias but not the other
  * **Niko**: it doesn't know there's aliasing there
  * **Niko**: reason you might want it, even if it's not default: sometimes you want aliasing, I dunno
  * **Niko**: not sure there's a really good reason
  * **Graydon**: implementation-wise: can I pass my `&mut T` to a sub-function?
  * **Niko**: you could reborrow it as `&mut`
  * **Graydon**: suppose we do most conservative thing. presumably `&mut T` -- there can only be one. so I can move it to a sub-function
  * **Niko**: yep
  * **Graydon**: can I get it back?
  * **Niko**: yeah. way it's modeled in type system is that you "re-borrow" it. may be implicit on calling function.

```rust
    fn foo(x: &mut T) { bar(x); x.foo = 22; }
    fn bar(x: &mut T) { ... }
```

  * **Niko**: that's the example you mean right?
  * **Graydon**: yes. if you take a value it expects to write to, you're gonna write that
  * **Niko**: that's okay, or we can make it okay. what's sort of happening is this:

```rust
    fn foo(x: &mut T) { bar(&mut *x); x.foo = 44; }
    fn bar(x: &mut T) { ... }
```

  * **Niko**: the type system thinks about it as re-borrowing x for the call. this is totally invisible unless there were some closure:

```rust
    fn foo(x: &mut T) { bar(&mut *x, || x.foo = 22; /* error */); x.foo = 44; }
    fn bar(x: &mut T, y: &fn()) { ... }
```

  * **Niko**: you get it back again when borrow terminates.
  * **Graydon**: okay, don't understand what part of type system that re-borrow is interacting with. creating a new lifetime?
  * **Niko**: that's right. well... 
  * **Graydon**: creating a new value and associating value with a new lifetime
  * **Niko**: a more limited lifetime. that's right. the new value is restricted to more limited lifetime
  * **Graydon**: borrowck already works that way
  * **Niko**: that's right. when you re-borrow you make the original thing inaccessible for lifetime of re-borrow
  * **Graydon**: so long as that works, I'm ok
  * **Niko**: when you think about it: what we do now when we auto-borrow parameters, this is exactly the same situation
  * **Graydon**: just wanted to make sure
  * **Brian**: what happens if bar wants to store or return the pointer?
  * **Niko**: that would work same way that works today with temporarily freezing something
  * **Niko**: if it were to return it, would have to link lifetime of parameter to lifetime of thing returned; that would extend the `&mut` borrow
  * **Niko**: extended to until the returned value goes out of scope
  * **Brian**: okay, I see
  * **John**: as user of `bar`, you need to know whether `bar` is going to store the parameter?
  * **Niko**: all that info is contained in the type signature. it would have to link the lifetime to the lifetime of something else it stores in
  * **Graydon**: off top of my head, I like all three prongs. seems like it tidies things up as inherited mutability as the way to think about mutability
  * **Patrick**: not sure how much auto-reborrowing will come up. `&mut self` is usually the self parameter. auto-reborrows automatically happen on LHS of dot
  * **Niko**: will have to change that mechanism a little bit to make it compatible with this. all you're saying is: we use this auto-reborrowing a lot with self-parameters but maybe not so much with others
  * **Niko**: as long as we keep assignability we'll keep this
  * **Niko**: defer everything else to planning meeting?
  * **Graydon**: effectively out of time
  * **Graydon**: anything else super high priority?
