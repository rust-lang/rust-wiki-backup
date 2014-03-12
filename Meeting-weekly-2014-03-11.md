# Agenda 3/11/2014

* unsafe ptrs, *mut (brson) https://github.com/mozilla/rust/issues/7362
* stronger guarantees for &mut borrows (nmatsakis) https://github.com/mozilla/rust/issues/12624
* robin hood hashtable https://github.com/mozilla/rust/pull/12081
* applying coercion rules on return (nmatsakis) https://github.com/mozilla/rust/issues/12755
* self argument destructuring (brson) https://github.com/mozilla/rust/pull/12566
* type hints (acrichto) https://github.com/mozilla/rust/pull/12764

# Attending

- nmatsakis, dherman, brson, pnkfelix, acrichto, pcwalton, larsberg

# Status
- nmatsakis: RFC writing
- brson: RFC, doc, install, probably other stuff
- pnkfelix: Control Flow Graph, RFC, misc landing
- acrichto: chan renaming, liblog, RFCs, switch to libnative, I/O bugs
- pcwalton: removing ~[T] and ~str from the language
- nrc: (on vacation)

# Robin hood hashtable

- brson: I r+'d it.
- nmatsakis: Great!

# stronger guarantees for &mut borrows

- nmatsakis: RFC changes the rules so you can't read things that have been &mut borrowed, even if we can see that it's safe. I've gotten only positive feedback, but haven't put it forward as if we want to do it or not.
- pcwalton: Would like to see how much it breaks.
- nmatsakis: I think you can only create this scenario within one function, so it should not be too much. I will have to implement it to find out, though. I will try to implement it and assuming it does not cause massive disruption, I will put it in.

# self argument destructuring

- brson: Got a PR that does this. Do we want to do this?
- acrichto: With UFCS, this support will just fall out
- brson: If you want to destructure, you just don't use the special self sugar
- brson: Then I will handle this.
- nmatsakis: I would not want any new syntax for this.

# type hints

- acrichto: Got a PR for _ as a type hint. It's a tiny change
- brson: Partial type hint: allows you to exclude type parameters.
- acrichto: Only allows you to do it within a function.
- brson: What does thi ssolve?
- pcwalton: Lots! Niko and I want this. Useful if you want to leave off type parameters. So, now, you can finally transmute like C++ without writing lots of type parameters. Methods like collect also make you say what you're collecting, which you now would no longer need to. Bunch of others like that.
- brson: Allows you to express things without as many words, but this doesn't seem like a major problem.
- nmatsakis: It fixes a wart...
- pcwalton: More fixes the papercut. Transmute is the biggest use case for this.
- nmatsakis: Also, I have an old blog post that explains why I'd like this (early/late-bound stuff). It opens up being able to omit lifetime parameters until the function is called, which we can't do today. Without going into the details, it's not just sugar.
- pnkfelix: Also, if you're doing collect on an iterator... (as pcwalton mentioned).
- nmatsakis: It also lets you just say the important type.
- brson: Can you write `Vec<_>`?
- acrichto: You can see it in the tests.
- brson: I was thinking `~[_]`
- nmatsakis: That should work, too.
- brson: Wait, removes return types? That seems super-powerful...
- nmatsakis: I think those are error cases.
- brson: Beside my normal fear about adding features, I have no complaints.
- nmatsakis: Let's do it.
- acrichto: Should we consider syntax? I'd like for casting & pointers for * pointers, but looks silly as `as *_`.
- nmatsakis: Why would you do that cast?
- acrichto: For runtime type things and FFI-type things you write it a lot.
- nmatsakis: Doesn't happen much.
- acrichto: larsberg runs into that...
- larsberg: The auto-generated DOM code runs into this frequently.
- nmatsakis: Maybe just write a wrapper function.
- acrichto: I was thinking ?, but that doesn't match. `_` is probably the right syntax.
- brson: Let's do it.

# coercion rules on return

- nmatsakis: Ordinarily, we apply type coercions any place where the type is known. Parameter passing, struct field assignments, and lets with a specified type... but not returns. I put in an issue with an example of some code that doesn't work though I feel like it should if we did this. See PR.
- brson: Why is that a coercion in the linked PR?
- nmatsakis: ... it's a corner case.
- brson: What are the examples other than &mut?
- acrichto: a ~T field and want to return an &T, need an as_slice, etc.
- nmatsakis: Probably still need it, because that's how our coercion rules work. Or if you want to return a Trait object and have a value instead and have to do `as T`.
- brson: Same routine that we use today?
- nmatsakis: Yes.
- brson: May even add more &s to make it work, for autoref, etc?
- nmatsakis: Only autoref on `.`. For now. I just want to use the same rules that we use on parameters, whatever they are.
- brson: You said this was non-controversial. Does anybody not want to do this?
- acrichto: It looks like you're moving out of a field, but that's how it works in the other cases. We should just try to be consistent.
- nmatsakis: Yes, most of the complaints seem to be around the coercion rules themselves.
- brson: Let's do it.

# unsafe ptrs, *mut

- nmatsakis: Not as easy as I thought. I also feel like our current situation is not great. I'm concerened because today you can write `*T` and `*mut T` but they have very different semantics. I can tell you what I think they are, but they don't match other Rust types or C types. `*T` is what C calls `const T *`; aliased pointer you're not supposed to mutate through, but can me mutated beneath you. `*mut T` is like C's `T *`, which you can both mutate and alias. No equivalent in the safe dialect of Rust.
- acrichto: Why do we have both?
- nmatsakis: Or, why would we want both? Might want readonly because it lets the compiler optimize more, potentially. 
- acrichto: Is that optimizing the caller or callee?
- nmatsakis: Caller. I haven't read the C standard in great depth, but I'm assuming the goal is to make the effects of mutation undefined, allowing the caller to perform optimizations assuming it doesn't change.
- acrichto: Seems like something we'd read in from header files instead of having in the type system.
- nmatsakis: It's part of the C typesystem. Also don't have `restrict` pointers, which are there to optimize the callee, because it lets the callee assume the memory does not overlap. Doesn't matter to the caller, because the caller presumably knows if the memory overlaps. If I were writing a Rust function, I would drop the restrict.
- brson: I feel like the optimizations enabled by `const T *` are not that important and this is just going to be extra baggage.
- nmatsakis: There is some sanity checking you get from the `const` keyword... in other words, if you have a `*` and try to pass it to something that takes a `*mut`, I would get a type error. But it feels like a choice that we can't walk back from. Once we release libc versions that don't accurately model the types that the C headers have, then for backcompat it's hard to think that we can model them with these features later without breaking APIs.
- brson: Is there much of that in the C stdlib?
- nmatsakis: I think so. But, assuming you don't change the return value, I think you can always change a `*` to a `*const`. 
- pnkfelix: Higher-order case?
- nmatsakis: Not a big deal for C functions... maybe qsort?  I do think you almost always want `*mut T` in our code. `*const` is more for temporaries that go into function parameters and so forth.
- brson: At least, we need to change `*mut` to `*`.
- nmatsakis: So, `*T` is the same as `T*`? I can get behind this.
- pnkfelix: Do we want two variants? If just one, then it should be `*T`. The interesting question is: do we want two at all? 
- nmatsakis: Not sure. Three possible schemes I like. `*T` by itself, which is exactly what C's `T*` means. There is `*T` for C and `*const T`. Third, `*T`, `*const T`, and `*mut T`. Doesn't make a lot of sense to use a different keyword than const (just copy C).
- acrichto: Could we add `*const` in a back-compat way later?
- nmatsakis: I think the answer is mostly.
- pnkfelix: I'm mainly concerned with higher-order functions that take a `*const`. I think the more interesting question is what LLVM might do to optimize your code. It's more about sanity-checking and safeguards, but I don't even know if we provide you any of that.
- acrichto: We don't have any proven wins from giving LLVM more info about raw pointers like that, so I'm tempted to add `*T` and just not worry if we break you.
- brson: Silent change?
- nmatsakis: No, things will stop compiling. One scenario is if we convert `*T` to `*const T` in a higher-order case, it could break user code that was providing a simple `*T` function to the function as an h-o argument.
- brson: Is const deep or shallow?
- nmatsakis: Shallow.
- brson: Compatible with aliasable references? Those are no longer deep...
- nmatsakis: Compatible... but not the same. It's a weaker guarantee. e.g., `&int` is immutable. `*int` is not. Also different w.r.t. members of a struct.
- brson: Does our compiler even have enough info to prevent you from violating the `*const` contract?
- nmatsakis: Yes.
- acrichto: How? You can cast raw pointers however you want!
- nmatsakis: Hrm. C works harder than we do to prevent us from casting away constness...
- brson: We'd have to add that in the first place; we don't have `*const` anyway.
- nmatsakis: We currently allow raw pointer casts for static initializer cases.
- brson: Can we prevent the "wrong" direction on these casts?
- nmatsakis: Yes. Then people can force it with transmute... it's appealing to just have `*T`.
- pnkfelix: Seems like a lot of commenters on the ticket want the two variants because it lets them accurately translate their C bindings. But, this discussion about the guarantees has made me worry that if we act like we support this like C but we don't in reality, it will be bad. 
- nmatsakis: An argument for having `*const` is that we allow implicit coercion from `&T` to `*T` and `&mut T` to `*mut T`. Seems unfortunate to allow implicit from `&T` (immutable) to new `*T` (mutable). That was huon's example:
```
extern "C" fn inc(x: *T); // 
fn foo() {
    let x = 3;
    unsafe { inc(&x); }
    // voila, x == 4, might be nice if &T -> *T were an invalid coercion, but &T -> *const T is valid
}
```
- nmatsakis: Kind of unfortunate that there is no warning here. Always possible if the C signature is wrong, but if we had `const` might be able to make this an invalid coercion.
- brson: So, Rust semantics don't allow this optimization... can we make this miscompile.
- nmatsakis: Certainly, a later reference to `x` might be replaced via constant propagation with 3, which would not be the expected behavior. Our inability to express as much as the C typesystem would prevent our ability to catch this, for better or worse.
- brson: That coercion seems really wrong to me, now that you point it out.
- nmatsakis: I was initially in favor of one pointer type, but this example is what put me back on the fence.
- acrichto: In theory the signature of the inc method should be *mut.
- nmatsakis: I agree that there are a lot of potential ways it could go wrong. Signature could be wrongly transcribed. But, if we only had one pointer type, this would be correct. We have a bug, but it's correctly transcribed. If `*T` were the only type we had and we keep this coercion, then it's just something you have to know and watch out for.
- acrichto: Can we forbid `&T` to `*T` and require `&mut T`? Arguably `*const` is still wrong because you can follow the inner pointer, which you can't do...
- pcwalton: Not so worried about this. It's in an `unsafe` block and you violated the promise. Wouldn't the analog to this in C be `const T*x`. So, you pass your reference to a function, but it casts away const. Also, could happen if you do `extern C void inc(const int *x);` but link to an object that takes a `int *`, which will compile...
- nmatsakis: But you could read the declaration and notice that. Here, there is nothing to see.
- pcwalton: You would notice it in C as well. 
- nmatsakis: But if we only have one pointer type, we would not be able to distinguish this.
- acrichto: Technically, this could be `&mut T`. It's not impossible to represent... it just doesn't look like an FFI function.
- nmatsakis: In other case, becuase `&mut` prevents aliasing, it might not be suitable.
- acrichto: `&mut` is probably stricter than any C function would require.
- nmatsakis: I'm concerned that for `Rc<Arc<>>`, which can't provide you an `&mut`. There's no way to express it in Rust.
- brson: Not bad if `&mut` coerces to `*T`, right?
- nmatsakis: If you can't get from `&T` to `*T`, that's annoying. So, either way have a way to say it in the type system and be stricter (the `const` approach) or we leave it to the user to validate that use of an `&T` is safe. Alex is right that because const is shallow, it's nothing more than a lint, really - no guarantees when you call C code. This is just a way to catch some possible bugs.
- brson: If there are no guarantees, doesn't that transitively make us unable to optimize things around `&T`?
- nmatsakis: It's undefined behavior to mutate the contents of a `*const`. Maybe another question here is to what extend, when we're interfacing with C code, do we get to act like an optimizing C compiler and where do we have to be conservative? I've been assuming we will act like a C compiler.
- pcwalton: I like that. C compilers do those optimizations for a reason and I'm worried that if we can't do that in our Rust code that interfaces with C that people will just drop to C instead.
- huon: auto-generated bindings ala rust-bindgen / https://github.com/mozilla/rust/issues/2124 would solve the "transcribe signatures wrong" problem (I forgot about the US DST switch, so I'm an hour late... whoops :( )
- pcwalton: Not with just one pointer type, though... so maybe let's keep `*T` and `*const T`?
- brson: I think that's what Niko wants.
- pcwalton: That's kinda like C... I like that...
- brson: I've been beaten down and am willing to accept this. I'd like to know what restrictions you want on reference coercions. 
- nmatsakis: `&T` coerces to `*const T`, `&mut` coerces to both.
- pnkfelix: Any deep analysis we can do on T to push constness through it?
- nmatsakis: If there are *-pointers in there, I think we're fine. It's just if you reach an & through a *-pointer that you reach a problem with the semantics. I kinda feel like we can't solve this. I'm not inclined to do anything more advanced than what we proposed.
- pnkfelix: Maybe a lint for an unsafe function that has a raw pointer with a & inside of it... weird.
- acrichto: Pointers inside of other pointers is not unheard of; all over the libuv.
- brson: Are they *-pointers?
- acrichto: Only mutable references, right now...
- nmatsakis: Not giving any guarantees; you still have to have some idea of what a function is going to do with the data you pass it.
- brson: Tentative agreement here. Are we happy?
- nmatsakis: I'm happy with `*T` and `*const T`.
- brson: Will you write the RFC?
- nmatsakis: Yes. 
- pcwalton: This will break Servo badly. And nearly all Rust code that uses the FFI.
