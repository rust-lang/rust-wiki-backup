# Agenda 11/19/2013
* Result api redesign (acrichto) https://github.com/mozilla/rust/pull/10364
* #[thread_local] (acrichto) https://github.com/mozilla/rust/pull/10312
* autoderef and args (acrichto) https://github.com/mozilla/rust/issues/10504
* libcxxabi/libunwind to drop libstdc++ (acrichto)
* static linking (acrichto)
* semantics of `_` in patterns (nmatsakis) https://github.com/mozilla/rust/issues/10488
* fate of 'self (nmatsakis) https://github.com/mozilla/rust/issues/10565
* lifetimes in bounds (nmatsakis)
* user-defined vecs (nmatsakis)
* task joining (pcwalton)

# Status
- nmatsakis: Lifetimes in bounds (#5121), freeze `&mut` in `&` (#10520), closures (#2202)
- brson: android test automation
- acrichto: static linking, native I/O, static libcxxabi
- pcwalton: proc reform (partially landed), &fn reform (not landed), "new" as keyword
- pnkfelix: GC, preparing for Codemesh talk

## static linking
- acrichto: Have an implementation. Summary: create rlibs, which are .o files (ish). With that, everything works. Can't mix dynamic & static libs now. In theory, possible, but it's tricky because you need to avoid having a library show up twice. Encompasses most use cases, especially making it possible to reduce the amount of stuff you ship.
- pcwalton: dynamic C libraries with static rust libraries?
- acrichto: Native libraries. Now two versions. The link attribute will have name=something and kind=something. For kind, will have static, framework (for OSX), etc. If omitted, propagated to the final target. If you create an rlib, you have to encode what your dynamic native dependencies are. We are not propagating link args across multiple crates. Only to your final target. Native static libraries are a little different and part of build processes, so want to avoid distributing them. For rlibs, you should pass -relocatable to ld, which will process a .o file, and will bundle native static libs so you don't have to distribute them.
- pcwalton: Can you link against dynamic shared libraries from a static rust executable?
- acrichto: Yes.
- pcwalton: Important because some libs you can't statically link against. Down the road, we'll need to mix dynamic & static rust libs, but fine for now.
-brson: Argument prop for native libraries here?
- acrichto: Yes. If you create an rlib, can't link dynamic native dependencies.
- brson: Pathes for -L flags, too?
- acrichto: Don't deal with them at all. Up to the final link step. We can't even change that. Could say, PATH=blah, but there's no way to put it in the executable right now. This also does not propagate the link_args attribute.
- brson: How do we test this?
- acrichto: New test infra. run-make. Have scripts that do run-make, uses ENV vars for doing a bunch of different permutations. Mainly testing symbol resolution, not just running things.
- nmatsakis: Can you reconcile the RLIB vs. static rust libs being able to link dynamic native dependencies?
- acrichto: If libfoo.rilb depends on libm, the rlib has not linked to libm. So when you link to libfoo.rlib in an executable, you pick up the -lm to pick up libm.
- nmatsakis: What does it mean to link against the dynamic lib in an rlib at all then?
- acrichto: Link flags.
- nmatsakis: Maybe later, what's the problem with mixing dynamic/static linking?
- acrichto: Will create an open issue with the details.
- brson: Changes to snapshot?
- acrichto: Nope. Just using the dylibs. No reason to link statically against librustc. We won't build/release static versions of anything other than libstd. Everything else dynamic.
- brson: Sounds amazing!

## wildcards in pattern matching
- nmatsakis: Treatment of `let _ = foo` drops foo, and the compiler doesn't recognize that. Because we have two interepentations of _ in the compiler. Two are: if you write `let _ = expr` then it means "consume and drop expr". If you have them in the pattern embedded, then ignore the item. Borrow treats it like ignore. Trans treats it like drop except in that special case. Ignore is always the right behavior. So, `let _ = expr` would not drop the expr. If you have an lvalue:
```
let x: ~str = ~"hi";
let _ = x; // no-op tomorrow, not today
let y: Option<~str> = Some(~"hi");
let Some(_) = y; // no-op today and tomorrow
```
- nmatsakis: Today, these two are distinct.
```
let _ = ~"foo"; // no-op from pattern matching, but temp will still get dropped
```
- nmatsakis: Now, will be dropped not because it's dropped but because it's a temporary. By the same mechanism, the rvalue (string literal) would still be dropped. Advantages: if you interpret _ consistently as drop:
```
let x: &SomeStruct = ...;
match *x { SomeStruct(_, ...) => { // ERROR -- _ is dropping from a borrowed ptr }
```
- nmatsakis: Couldn't use _ inside of a borrowed pointer. Feel like this is a good motivator.
- pcwalton: Agree. We should have an ignore function and put it in the prelude.
- nmatsakis/jack: Could call it drop.
- pcwalton: Sounds good, and less characters.
- nmatsakis: doesn't ignore it's argument. If you write `ignore(x)`, `x` is not ignored.
- brson: Wait! So these moves would become not moves if you have an _ in the pattern?
- nmatsakis: Depends on if you bind. If you bind by value, then it's a byval move/copy (depending on type). If you bind byref, it's always creating a pointer to the original. So, _ is equivalent-ish to a ref you never use. Requires less from the borrow checker because you never use it.
- pcwalton: Want _ to drop things. Because you want to.... <breaking up>
- nmatsakis: If _ drops, then there are places you can't use it. Current way to drop is to bind with a random variable name.
- acrichto: Wary of adding drop to the prelude, but other than that it's perfectly fine.
- nmatsakis: Separate thing anyway.
- nmatsakis: could call it `free`?

## Task joining
- pcwalton: strcat mentioned a problem implementing the rust task API on top of the pthreads API. Semantic mismatch between the two. In pthreads, two mismatches. First (important one): in pthreads, when you spawn a task, you get a task handle and can join on it later. That waits for it to exit (Can also receive a value). If you don't want that, then you have to call `pthread_detach`. We don't support joining in pthreads, which seems unfortunate because if you want an impl on top of pthreads, you're not going to have this facility, which limits porting pthreads programs. Having join also prevents the pattern in servo of sending an exit message to a task and then blocking on the response.
- brson: You do have a function that tells you when the task ends...
- pcwalton: block on it?
- brson: Yes, it's just a message. Just request it when you launch the task.
- pcwalton: Maybe no mismatch, then.
- nmatsakis: I always forget to call detach, get memory leaks, etc.
- pcwalton: Maybe bad defaults?
- nmatsakis: Just saying that I never use join, though I know it's good for fork-join style code.
- pcwalton: The channel/port thing in servo is kind of a botch.
- acrichto: People don't like the I/O blocking is unconditional and can't select on it. I'm wary of being unable to block on it. With channels, you can wait on it. A small tweak of the task API with the exit message (something that gives you a port) is great.
- pcwalton: Just want to have the functionality. strcat invented his own incompatible rustcore API, which we should strongly discourage.
- nmatsakis. Those two methods are functionally equivalent, but I don't think you'd implement one on top of the other.
- brson: Unless a separate fork type. But the easiest way is just to make all pthreads detached under the hood.
- pcwalton: My concern is to have rustcore & libstd not have incompatible APIs here.
- brson: Would like a better Task API anyway.

## autoderef
- acrichto: We have autoref/autoderef, but it's inconsistent. In the bug, if you have a function w/ a borrowed pointer and give it an owned pointer, it'll auto-whatever and make it work. But if you pass a value, it doesn't work. Kinda weird. Also lots of places where we don't do autoref/autoderef. If return is a borrowed but you return an owned string, you have to call `as_slice`. So, globally, autoref/autoderef are wonky. You have to already know where they will work. Should we change rules? Should it be a hierarchy of values, so you can always auto-borrow down the hierarchy? But, if you can auto-create borrowed pointers, can you also create borrowed mutable pointers? 
- nmatsakis: Glad this came up. Summarize as: we are inconsistently magical. Lots of magic in `.` and `[]`. A little around fun args, not much elsewhere. We should stop having magic around fun args and limit magic to `.` and `[]`. But, if we went the other direction, maybe it would yield a language with many less `&`s. But, you can't read a function in isolation. The current rules date from a different understanding of the language. `T` and `~T` was neither true nor recognized yet. I don't get surprised that I can't pass a T to an &T these days, but I do get surprised when:
```
let x = ~HashMap ();
foo(x); // I expect x to be moved here, but it's not necessarily
fn foo(x: &HashMap) { ... }
```
- nmatsakis: Need to look at `foo` to understand if there's a move or not. Don't have a strong opinion, but we're in a bad spot today.
- pcwalton: Remove all auto-borrow except ...
- acrichto: Sure, but then hard to get a borrow from a owned pointer
- all: `&*`
- acrichto: Ugly.
- nmatsakis: `&mut *` is ugly but the alternate bothers me very much because it hides mutation where I don't expect it:
```
let mut x = ~[...];
sort(x); // sorts x in place (!)
fn sort(x: &mut [ ... ]) { }
```
- pcwalton: Very strong argument. Fact that it mutates x in place is weird.
- pnkfelix: In favor of `& mut*` or not? lbergstrom can't type...
- nmatsakis: I'm used to `& mut *`, but every time I show it to smart folks, I have to explain it.
- pcwalton: autoborrowing also requires it.
- pnkfelix: Sometimes I've had to use `&*`, but since you don't always have to use it, it's a challenge.
- nmatsakis: Interaction between inference and auto-borrowing that prevents us from doing it all the time.
- acrichto: I don't like `&*` but do like `&`. Weird to have magic from `&~` to `&`?
- pcwalton: No.
- acrichto: If you have `~Foo` and the function wants ...
- pcwalton: With generics, I don't think it will work.
- nmatsakis: Also possible to pass in a pointer to a unique pointer. 
- pnkfelix: Will learn a lot from the exercise of putting in the `&*`s.
- nmatsakis: Proposed from time to time on #rust, and basically nobody is in favor of less magic.
- pcwalton: This changes makes sense, though. We've always regretted magic. So I'm by default in favor of non-magical designs. Especially w.r.t. memory management. 
- nmatsakis: Maybe something that's less ugly than `&*` syntactically?
- acrichto: Compelling to remove. Let's post to the mailing list with justification and see if there's a good suggestion.
- pcwalton: Sounds good and might give them a nice heads-up.
- acrichto: I'll do it.

## user-defined vecs
- nmatsakis: Most of my thoughts are in the blog post. In short, another way we could do it without DST (dynamically sized types). Still need the other blog post to see if it removes the need for DST entirely -- I think not. Some advantages / some disadvantages.
- pcwalton: Your proposal was a bit farther than I'd thought. I envisioned fixed-length vectors in the language and fundamental. In a static, a vector literal is always fixed-length. Vector literals in non-static contexts could resolve based on inference. 
- nmatsakis: Would resolve the static question. If you do this builder approach I talked about for literals, then there's allocation that is not entirely obvious. 
- pcwalton: OK with punting on literal syntax. Just say that if fixed-length vectors are the builtin (which interacts strangely with removing auto-borrowing from fixed-length to slice), then it's fine to say small_vec::init(fixed-length vector) auto-borrows to a slice. Then it's explicit and just as efficient as having some sort of literal syntax and a trait. And generalizes to hash_maps.  Further, no problems w/ statics because can't have a small vector in a static due to dynamic allocation.
- nmatsakis: Appealing. Have the current syntax. It's always fixed-length, just like today, but we remove all the other syntaxes.
- pcwalton: Yes, and then use constructors to get the same result.
- nmatsakis: Maybe a macro that expands out.
- pcwalton: If we need special literals, could introduce them later. Doesn't seem that bad. Going to write x=small_vec::init() and add a type anyway since you need a type hint.
- pnkfelix: When you say small_vec::init of a fixed-length vector, is the argument an expression that should be valid in a static context? Or something that works in a dynamic context? What exactly are we punting and proposing?
- pcwalton: Punting on literal syntax being overloadable. So literal syntax for vectors always yields a vector.
- pnkfelix: stack-allocated fixed-length vector?
- pcwalton: Yes, and read-only. If you want dynamic allocation, need an explicit constructor form using the fixed-length vector as the base.
- nmatsakis: Others in the room? This is a pretty big change. 
- acrichto: summary?
- nmatsakis: Could remove vectors from the language proper and make them a library. In concrete, the types change as:
```
    ~[T]  ==>  Vector<T> (or maybe [T])
    &'a [T]  ==> Slice<'a, T>
    ~str ==> String
    &'a str ==> Substring<'a>
    [T, ..N] ==> [T, ..N]
```
- nmatsakis: And for expressions:
```
    let x = ~[1, 2, 3];  ===> let x = Vector::init([1, 2, 3]);
```
- acrichto: Is this essential for GC'd vectors? Can you do that at all with our current ones?
- nmatsakis: With DST, you would probably be able to write:
```
GC<[T]> <-- single indirection
GC<Vector<T>> <-- double indirection
GC<~[T]> <-- 
```
- nmatsakis: Means " a vector allocated by the GC". The vectorness and the allocations can get tied together. Not clear how much of a problem the indirection approaches are.
- pcwalton: Could have a kind of GC-vector together if one wanted.
- acrichto: Would have to re-implement vector entirely.
- pcwalton: Set of ops on a unique vector very different from a GC vector (e.g. push in place due to no aliases on unique).
- nmataskis: Narrows when DST is important.
- pcwalton: small vector is the big argument for me. In servo, optimizing pretty heavily, would like it first class.
