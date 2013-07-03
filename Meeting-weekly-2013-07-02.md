# Agenda

    - 0.7 release status
    - Fate of @mut
    - @ constructor bounds / &-and-dtors

# Status of 0.7 release

- brson: Struggling to get release candidate tarball presently
- brson: Android build fixed over weekend
- brson: Rusti failures in cross compilation; bots are running 'check-lite' on cross compile

# Fate of @mut

- P: Discussed last week, not enough people to discuss fully
- P: Possibly remove @mut, just use Cell or something
- P: Rationale: the write barriers are pretty much invisible, appear in pattern matches etc, very surprising
- P: Leaning towards just Cell, even though it combines null-ness and borrowed-ness; the effect to the programmer is quite similar: "value not accessible"
- P: if we move this into pure libraries, no compiler support is present at all
- N: there are three approaches: (1.) dynamic borrow, (2.) RAII, (3.) closures, ...
- P: maybe .. there's a sub-topic, the @-constructor

# `@` constructor

- P: currently there are no restrictions placed on "what can be placed inside @-box". Not true for @Trait and @fn, but normal @ is unconstrained
- P: in particular you can put region pointers inside
- P: this makes it difficult to do a particular pattern with destructors
- N: region analysis only understands lifetime of reachability, assumes that if an @-box goes out of scope it's inaccessible; but if there's a dtor it may still access the &-var during destruction when the GC gets around to it.
- N: our current solution to this problem is to limit what _dtors_ can access. But this is only one way to solve the problem.
- Bblum: you still need some restrictions on dtors even if you fix this
- N: the danger ben is referring to is that if you have 2 objects that refer to one another, ...
 
```rust
     //Two objects in the same scope that both have dtors:
     {
        let object1 = ...;
        let object2 = ...;
        object1.field = &object2;
        object2.field = &object1;
     }
     //Bad.
```

- G: I think we're not supposed to permit destructors on cyclic values at all, no?
- N: if we prohibit & and @ in objects with dtors, that's true.
- N: but it rules out some desirable things. it would be nice to be able to do:

```rust
     fn foo(rwarc: &Rwarc<T>) {
        let write_lock = rwarc.write_lock();
        write_lock.contents = foo; 
     }
```
(the above example is using RAII to automatically release the write_lock.)
(RWARC already uses unsafe destructors like this; this would just allow the implementation to avoid #[unsafe_destructor].)

- N: Similarly, the use of `Cell` would be the same.
- P: this is not a theoretical limitation, this is something we run into all the time in Servo due to use of custom smart pointers for the DOM.
- P: the way we avoid this currently is to use closures, but that produces lots of rightward drift
- N: in case it's not obvious, the RAII-object needs to hold an &-ptr on the lock
- N: so, suppose we take advantage of lifetimes: it's ok for objects with dtors to have &-ptrs, but they must be longer than the object lifetime itself.
- G: yeah, that's what I'd be suggesting.

```rust
let val1 = ...;
let val2 = HasDtor { v1: &val1 }; // illegal
```

- P: everyone's surprised that you can put & in @
- B: I still don't see why we need it
- (G: I don't know either!)
- N: partly, it takes extra type rules to make it not-work, and it seems harmless
- N: I think the other use-cases are marginal, and you can get by with ~
- N: eg. sometimes you want a function that returns an @Trait that closes over &-ptrs, say. Like in the case of iterators say, or parser combinators. But it only really applies when you're trying to hide implementation details, and in those cases you can use a ~Trait.

```rust
// Example that is not totally unreasonable:
fn mk_reader<'a>(s: &'a str) -> @Reader:'a {
    BufReaderImpl {s: s} as @Reader:'a
}

// But it can also be written:
fn mk_reader<'a>(s: &'a str) -> BufReaderImpl<'a> {
    BufReaderImpl {s: s}
}

// Or it could be written, if you really wanted to hide impl details:
fn mk_reader<'a>(s: &'a str) -> ~Reader:'a {
    ~BufReaderImpl {s: s} as ~Reader:'a
}
```

- G: ok so the only set of cases you have in mind are those with @Trait (since they need bounds)?
- N: Somewhat. Consider second example above. Reveals implementation details, but also in some ways preferable: no heap allocation, can inline, etc.
- N: Using ~ loses the aliasability of @, but you can also just borrow the ~ to get aliasability.

```rust
// Legal today:
fn foo<A>(x: A) -> @A { @x }

// Illegal under this proposal, so you would need
// 'static to guarantee that type A doesn't have borrowed
// pointers, and hence can be managed:
fn foo<A:'static>(x: A) -> @A { @x }
```

- G: so .. to get a bit concrete, what's being proposed?
- N: 1. limit what goes in a @box, require any &-ptrs to be 'static
- N: 2. modify dtors to permit &-ptrs in the destructed object whose lifetime is strictly greater than the lifetime of the object.
- Bblum: this sounds surprising. You could define an object that has a &'self ptr in it that can never be instantiated.
- N: the only situation that would be ruled out would be if you want something with a dtor that points to something you declared earlier in the same block, and you want dtor to run on exit from the block.

```rust
// Would not type check because `lock` contains a pointer to
// `locked`, and that has the same time as lock:
{
let locked = RwArc { ... };
let lock = locked.write_lock();
lock.foo;
}

// Would need to introduce a block:
{
    let locked = RwArc { ... };
    {
        let lock = locked.write_lock();
        ...;
    }
}
```

- G: how much work would it be to modify lifetime rules so that first works? just compiler munging, right? If we're trying to copy C++ idioms, we should try to aim for #1 there.
- N: no theoretical difficulties, just have to tweak how hierarchy is defined
- G: this seems reasonable to me, I tend to have trouble comprehending the use cases for borrowed pointers in managed data anyhow.
- N: another advantage of this is that it absorbs a number of use cases for once fns. Not the task once-fns but the closure kinds. No rightward drift.
- P: yeah, writing the closures we're writing in servo is ... (rightward drift?) Also slightly worried about compile performance.
- N: ok, so sounds like generally indifferent-to-positive opinions?
- F: so we wouldn't have lifetime bounds on @Trait anymore?
- N: I hadn't planned to change how lifetime bounds _work_, you could write it it would just probably be an error further on. ~Trait:'lt still makes sense.
- N: Cf. exmple above. If you really wanted to hide implementation detail, not sure how else you'd do it. I guess you could take a closure.
- J: is the mechanism that would be used to constrain the contents of @Trait objects also be applicable to user-defined smart-pointers?

```rust
impl<A:'static> GC<A> {
    fn new(a: A) -> GC<A> { ... }
}
```

# Back to @mut

- P: With that decided, we could replace the methods for accessing @mut concents with an RAII system, like:

```rust
// Long hand access:
    {
        let as_mut = box.as_mut();
        as_mut.field = ...;
    }
```

```rust
// I think this would also work:
box.as_mut().field += 1;
```

- P: to be concrete:

```rust
let box: @Cell<int> = ...;
box.as_mut().value = 10;
```

- G: does this have to be an `@Cell`?  (i.e. One could just write "Cell<int>", right?)
- N: Yeah, this is like the old `Mut<>` structure.
... some discussion...
- N: yeah, what does `as_mut` return?
- P: some kind of & that can set the write barrier flag?
- P: has a pointer to the value and pointer to the write barrier
- P: flips the write barrier back on destruction
...
- P: want a mechanism for a managed pointer to a field of a managed object.

- G: Sorry to keep foot-dragging but .. I don't understand how `Cell` actually works
(also it's pretty difficult to take notes given that state!)

```rust
struct MutCell<'a, T> {
    priv cell: &'a Cell,
    value: &'a mut T
}

impl<T> Cell<T> {
    fn as_mut(&'a self) -> MutCell<'a, T> {
        self.lock_for_mutability();
        MutCell {cell:self, value: transmute(&self.data)}
    }
}

impl<'a> Drop for MutCell<'a> {
    fn drop() {
        self.cell.unlock_from_mutability();
    }
}
```

- G: this says nothing about GC
- N: the borrow checker will guarantee that the @-box is rooted for the lifetime of any &-ptr pointing into it
- G: To what extent does this feel like pushing unsoundness to a library? This is the same as what we have now?
- N: Same but more flexible, more explicit, and less magic
- S: feels less magical. It's documented and such. In a library. Obvious there are possible failure modes.
- P: in order to trip the write guard, you have to call a function. You might actually read the docs for that function.
- N: also integrates somewhat better with multiple smart pointer types. If you wind up with multiple smart pointers that have a mut and non-mut variant, they all have to support this functionality and do it correctly.
- B: Cell is a nice name, but it's kinda weird that this has nullability.
- G: if it's mutable, can't we say Cell<Option<T>> covers nullability?
- P: we could take out the nullability of Cell today.
- G: this doesn't make it much bigger, right?
- N: there's probably some overhead
- B: wouldn't want to change all existing code that does use nullability
- N: question: does everyone think Cell will continue to be useful?
- P: it's probable / possible that nullable-cell will not be useful.
- G: Why do we think the usefulness of it will change?
- N: it's mostly used to work around problems with missing once functions when writing tasks etc.
- Bblum: there's a use-case involving 2 closures that might take the cell, but not both.
- G: 2 mins left. Maybe naming? Cell and NullCell? Cell and Cell<Option>? Maybe just remove nullability "someday in the future"?






