This is more of a meta-proposal, in which I gather all the different ways we might fix the mess that is closures in the current system. There are a bunch of features we want that are sometimes at odds.

## Motivation

### Once-ness

We want library functions to be able to express that an argument closure will be called at most once (#2549). This will let callers capture noncopyable values and move them out in the closure body (e.g., sending an ARC endpoint over a pipe to another task).

### Environment kind bounds

Closures behave differently / have different capabilities depending on the kind bounds satisfied by the stuff in their environment. Closures with ```Const``` environments can't be used for the soundness exploit described in the next subsection. Closures with ```Owned``` environments (that are furthermore allocated on the ~-heap) can be used to spawn tasks. Closures with ```Copy``` (```Clone```) environments can be duplicated and used many times (as in #2830).

### Soundness with the new borrow-check

Now that ```&mut T``` borrowed pointers are noncopyable, there is a soundness bug as demonstrated [here](http://smallcultfollowing.com/babysteps/blog/2013/04/30/the-case-of-the-recurring-closure/). The gist is that, since closure environments are "opaque", a noncopyable mutable borrow can be captured in a closure, and then the closure can be copied somewhere the borrow information is not known.

(aside: This was not a problem for accidentally copying other noncopyables, since they could never be captured by-value, only "implicitly borrowed", which is why it wasn't legal to move them out.)

### Soundness with first-class dynamically-sized types

The recent proposal for unsized types (#6308) intersects the above soundness issue in a perilous corner-case, as explained [here] (http://smallcultfollowing.com/babysteps/blog/2013/05/13/recurring-closures-and-dynamically-sized-types/). If ```&fn()``` can be promoted to ```&T```, it can be copied unconditionally, which is a problem if ```&fn()```s can be called to mutate their upvars.

(That's a carefully-stated condition, because there are a number of ways to prevent this: ```&fn()```s can't be promoted, ```&fn()```s can't be called, ```&fn()```s can't mutably borrow their environments... each with different details and different degrees of feasibility.)

### Custom smart pointers

pcwalton has been thinking about custom smart pointers a lot recently. It would be nice if the language did not bless `@` and `~` in any way in regards to ability to create environments.

## Proposals

There are a bunch of ways to clean up the closure situation. The way I see it, there are 4 major decisions to make, all orthogonal to each other. I list them in rough increasing order of controversiality.

### I. Environment bounds

1. Default bounds are enforced based on the type of the closure (as today). Stack closures can have lifetimes; heap closures can only close over ```Owned``` data; neither type of closure can ever be ```Const``` (_as today_).

2. Function types can list, and hence inherit, kind bounds, as in ```~fn:Const+Owned()``` (such a function could be put inside of an ARC). Lifetime bounds can also be written, which would be useful for combinator-/curry-style functions, as in ```fn foo(x: &'a MyState) -> @fn:'a() -> int```. (#3569)

### II. Onceness

1. No oneshot closures, those passed to higher-order functions can be called arbitrarily many times, and captured upvars can't be moved-out (_as today_). To move-in/move-out, extra library functions would have to be written to take and pass an extra argument (```do rwarc.read(val) |val, state| { ... }```). The con here should be obvious.

2. Oneshot functions are written ```once fn()```. These are consumed when called, and of course noncopyable. The con is an extra keyword.

3. Closures are always consumed when you call them. Non-oneshot closures are indicated by ```fn:Copy()``` (later, ```fn:Clone()```). This is "theoretically nicest" -- it just works from the rest of the theory with no extra rules -- but has an uglier "default scenario" than case 1 (that is, all iterator functions, etc, must write kind bounds on their functions).

### III. Implicit vs explicit borrows in the environment

1. Stack closures can implicitly borrow; heap closures can't (_as today_). To put a borrowed pointer in a heap closure (assuming option I-2), you have to make another variable that explicitly borrows.

2. Replace implicit borrows with explicit ones, akin to ```ref``` pattern-bindings, introducing the option to have a capture clause. This makes it clearer when the closure wants to take ownership of something and how long closure-borrows last, and eliminates the "can't move out of captured upvar" error. The downside is referenced upvars get more verbose when autoderef doesn't apply (e.g., ```let mut x = 0; do something |ref x| { (*x)++; }```).

### IV. Fix the soundness bug.

In all of these cases (except B1), ```&fn()``` becomes noncopyable, unless it has a ```:Const``` bound. ```@fn()``` is a big problem if its environment is mutable, so any proposal that keeps ```@fn()```s (A4, B1, B2, B3, B4) would have to add a dynamic "recursion barrier" akin to the write barrier on ```@mut T``` boxes (not needed on a ```@fn:Const()```).

**A. Make closure types statically-sized.**

In all of these proposals, heap closures can't be promoted to borrowed closures. I don't know of any demand for doing so, and in any case you can always just do the manually-currying thing of ```|args| { heap_closure(args) }```.

1. ```&fn()``` becomes ```fn()```, ```@fn()``` is killed, and ```~fn()``` becomes ```proc()``` (variant 2 of [this idea] (http://smallcultfollowing.com/babysteps/blog/2013/05/14/procedures/)). The con here is that it breaks the "no dynamic allocation unless you see a ~ or a @" rule.

2. ```&fn()``` becomes ```fn()```, ```@fn()``` is killed, and ```~fn()``` becomes ```~proc()``` (variant 3 of [this idea] (http://smallcultfollowing.com/babysteps/blog/2013/05/14/procedures/)). Semantically this is no different from case 1, but the con is that ```fn``` is a first-class (sized) type, while ```proc``` is a second-class (unsized) type, which I think has a lot of confusingness potential.

3. ```&fn()``` becomes ```fn()```, ```@fn()``` and ```~fn()``` are killed and replaced by macros that use ```@Trait``` or ```~Trait``` to accomplish the same thing (especially in task spawning) ([this idea] (http://smallcultfollowing.com/babysteps/blog/2013/05/30/removing-procs/)).

4. ```&fn()``` becomes ```fn&()``` (```fn()``` can be sugar for this), ```@fn()``` becomes ```fn@()```, and ```~fn()``` becomes ```fn~()```. This retains most expressivity, but the con is that "common wisdom" says pointer sigils after types confuse people. (I don't mind; to me it suggests "I'm not a pointer but I have one inside". It's just parameterising over pointer types.)

**B. Make closure types dynamically-sized ("always have to be behind a pointer"), but prevent them from being copied.**

1. Borrowed closures can't be called at all as ```&fn()```; you have to use ```&mut fn()```. This has a horrible "default case", and also prevents oneshot closures from working (since you could just re-borrow the ```&mut``` to call it twice in a row, even though Y-combinator-like recursion is prevented).

2. Allow unsized promotion to ```&T```, but restrict when generic ```&T``` can be copied. This could be done with a new kind bound, such as ```fn copy_ref<T: NoMutableEnvironment>(x: &'a T) -> (&'a T, &'a T)```. This allows oneshot closures, but makes ```&T```'s behaviour depend on ```T```.

3. Don't allow closures to be promoted to ```&T``` (although vectors still can be). The con is the weird discrepancy between vectors and closures ("why can I do... but can't...?"). Note that this is almost identical to case 4, but the pointer sigils stay in front instead of behind the ```fn``` (except this still lets ```~fn()``` be promoted to ```&fn()```).

4. Don't do dynamically-sized types at all. This also saves the pain of introducing ```Sized``` bounds into all data structure functions, but of course precludes the future possibilities discussed in #6308.

## Ben's recommendations

**Ultimate preferences**

*I:* 2 for sure.

*II:* prefer 3; 2 would be OK.

*III:* prefer 2; 1 would be OK.

*IV:* in order of preference: B3, A4, A3, B4, B2.

**Scheduling for 1.0 vs post-1.0**

*I:* For the most part, we could leave option 1 and add option 2 post-1.0. However, for backwards-compatibility, we have to make sure that the "default bounds" in option 1 are "no bounds at all" (e.g., if v1.0's ```~fn()``` means ```~fn():Owned``` with post-1.0-option-2, then some v1.0 code that uses ```~fn()```s might assume ```Owned``` capabilities and break later). Since task body closures need to own their environment, we can't make no default bounds on ```~fn()```s, so either:
* We decide between option 1 or 2 pre-1.0 and stick with it forever
* We get rid of all heap closures (Niko's idea, IV-A-3), and can then make all default environment bounds empty.
However, note they add a piece of expressivity that we don't yet have (we can't put bounds on a Trait's existential data yet either).

*II:* Options 1 (nothing) and 2 (```once```) are future-proof (that is, option 2 can be safely added post-1.0 if we reserve ```once```). Option 3 (making ```fn:Clone``` mean not-once) is not future-proof, because it removes capabilities from plain ```fn()```s, so it requires both itself and option I-2 to happen pre-1.0.

*III:* Changing from either option to the other is not future-proof, no way, no how.

*IV:* Niko's idea, A3, can happen for 1.0 and later be upgraded to A1, A2, A4, or B3. Option B4 can be upgraded later to option B3 (that is, deferring dynamically-sized types to post-1.0). The other options are stickier.

**Recommended plan:**
* For bounds and oneshots: Because of my preference for II-3, and the note about expressivity, I want I-2 and II-3 to happen pre-1.0. If II-2 outvotes II-3, then (assuming IV-A3) I-2 and II-2 can both happen independently either pre- or post-1.0.
* For environment borrowing: I think III-2 is theoretically prettier, but it is a lot of churn, so I won't be offended if we do III-1.
* For fixing the soundness bug: IV-A3 is clearly the most flexible, both for rushing to 1.0 and for getting a nice final product.