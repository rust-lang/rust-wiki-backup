## General language issues

### I'm impatient. Can you give a brief summary of the salient features?

#### Safety oriented

* Memory safe: no null pointers, wild pointers, etc
* Expressive mutability control. Immutable by default
* No shared mutable state across tasks
* Dynamic execution safety: task failure / unwinding, trapping, RAII / dtors
* Safe interior pointer types with lifetime analysis

#### Concurrency and efficiency oriented

* Lightweight tasks (coroutines) with expanding stacks
* Fast asynchronous, copyless message passing
* Optional garbage collection
* All types may be explicitly allocated on the stack or interior to other types
* Static, native compilation using LLVM
* Direct and simple interface to C code

#### Practicality oriented

* Multi-paradigm: pure-functional, concurrent-actor, imperative-procedural, OO
 * First-class functions, cheap non-escaping closures
 * Algebraic data types (called enums) with pattern matching
 * Method implementations on any type
 * Traits, which share aspects of type classes and interfaces
* Multi-platform. Developed on Windows, Linux, OS X
* UTF-8 strings, assortment of machine-level types
* Works with existing native toolchains, GDB, Valgrind, Instruments, etc
* Rule-breaking is allowed if explicit about where and how

### What does it look like?

The syntax is still evolving, but here's a snippet from the hash map in core::send_map.

```none
    struct LinearMap<K:Eq Hash,V> {
        k0: u64,
        k1: u64,
        resize_at: uint,
        size: uint,
        buckets: ~[Option<Bucket<K,V>>],
    }

    enum SearchResult {
        FoundEntry(uint), FoundHole(uint), TableFull
    }

    fn linear_map_with_capacity<K:Eq Hash,V>(capacity: uint) -> LinearMap<K,V> {
        let r = rand::Rng();
        linear_map_with_capacity_and_keys(r.gen_u64(), r.gen_u64(), capacity)
    }

    impl<K:Hash IterBytes Eq, V> LinearMap<K,V> {

        pure fn contains_key(&const self, k: &K) -> bool {
            match self.bucket_for_key(self.buckets, k) {
                FoundEntry(_) => true,
                TableFull | FoundHole(_) => false
            }
        }

        fn clear(&mut self) {
            for uint::range(0, self.buckets.len()) |idx| {
                self.buckets[idx] = None;
            }
            self.size = 0;
        }

    ...
    }
```

### Does it run on Windows?

Yes. All development happens in lock-step on all 3 target platforms. Using mingw, not cygwin.

### Are there any big programs written in it yet? I want to read big samples.

There aren't many large programs yet. The [Rust compiler][rustc], 50,000+ lines at the time of writing, is written in Rust. A research browser engine called [Servo][servo], 17,000+ lines, will be a large task oriented application, integrating many crates and native library bindings (note that it can be difficult to build currently).

Some examples that demonstrate different aspects of the language:

* The core library's [LinearMap] - A sendable hash map in an OO style
* Servo's [image_cache_task] - An image cache in an actor style
* [fempeg] - An mpeg-2 decoder that does only stack allocation, no heap, no GC
* [mre] - A small web framework integrating a number of interesting bindings

[rustc]: https://github.com/mozilla/rust/tree/master/src/rustc
[servo]: https://github.com/mozilla/servo
[LinearMap]: https://github.com/mozilla/rust/blob/master/src/libcore/send_map.rs
[image_cache_task]: https://github.com/mozilla/servo/blob/master/src/servo/resource/image_cache_task.rs
[fempeg]: https://github.com/pcwalton/fempeg
[mre]: https://github.com/erickt/mre

### Have you seen this Google language, Go? How does Rust compare?

Yes.

* Rust development was several years underway before Go launched, no direct inspiration.
 * Though Pike's previous languages in the Go family (Newsqueak, Alef, Limbo) were influential.
* Go adopted semantics (safety and memory model) that are quite unsatisfactory.
 * Shared mutable state.
 * Global GC.
 * Null pointers.
 * No RAII or destructors.
 * No type-parametric user code.
* There are a number of other fine coroutine / actor languages in development presently. It's an area of focus across the PL community.

### I like the language but it really needs _$somefeature_.

At this point we are focusing on removing and stabilizing features rather than adding them. File a bug if you think it's important in terms of meeting the existing goals or making the language passably usable. Reductions are more interesting than additions, though.

## Specific language issues

### Is it OO? How do I do this thing I normally do in an OO language?

It is multi-paradigm. Not everything is shoe-horned into a single abstraction. Many things you can do in OO languages you can do in Rust, but not everything, and not always using the same abstraction you're accustomed to.

### How do you get away with "no null pointers"?

Data values in the language can only be constructed through a fixed set of initializer forms. Each of those forms requires that its inputs already be initialized. A liveness analysis ensures that local variables are initialized before use.

### What is the relationship between a module and a crate?

* A crate is a top-level compilation unit that corresponds to a single loadable object.
* A module is a (possibly nested) unit of name-management inside a crate.
* A crate contains an implicit, un-named top-level module.
* Recursive definitions can span modules, but not crates.
* Crates do not have global names, only a set of non-unique metadata tags.
* There is no global inter-crate namespace; all name management occurs within a crate.
 * Using another crate binds the root of _its_ namespace into the user's namespace.

### Why is failure unwinding non-recoverable within a task? Why not try to "catch exceptions"?

In short, because too few guarantees could be made about the dynamic environment of the catch block, as well as invariants holding in the unwound heap, to be able to safely resume; we believe that other methods of signalling and logging errors are more appropriate, with tasks playing the role of a "hard" isolation boundary between separate heaps.

Rust provides, instead, three predictable and well-defined options for handling any combination of the three main categories of "catch" logic:

* Failure _logging_ is done by the integrated logging subsystem.
* _Recovery_ after a failure is done by trapping a task failure from _outside_ the task, where other tasks are known to be unaffected.
* _Cleanup_ of resources is done by RAII-style objects with destructors.

Cleanup through RAII-style destructors is more likely to work than in catch blocks anyways, since it will be better tested (part of the non-error control paths, so executed all the time).

### Why aren't modules type-parametric?

We want to maintain the option to parametrize at runtime. We may make eventually change this limitation, but initially this is how type parameters were implemented.

### Why aren't values type-parametric? Why only items?

Doing so would make type inference much more complex, and require the implementation strategy of runtime parametrization. While this is our default implementation strategy, we don't want to mandate it.

### Why are enumerations nominal and closed?

We don't know if there's an obvious, easy, efficient, stock-textbook way of supporting open or structural disjoint unions. We prefer to stick to language features that have an obvious and well-explored semantics.

### Why aren't channels synchronous?

There's a lot of debate on this topic; it's easy to find a proponent of default-sync or default-async communication, and there are good reasons for either. Our choice rests on the following arguments:

* Part of the point of isolating tasks is to decouple tasks from one another, such that assumptions in one task do not cause undue constraints (or bugs, if violated!) in another. Temporal coupling is as real as any other kind; async-by-default relaxes the default case to only _causal_ coupling.
* Default-async supports buffering and batching communication, reducing the frequency and severity of task-switching and inter-task / inter-domain synchronization.
* Default-async with transmittable channels is the lowest-level building block on which more-complex synchronization topologies and strategies can be built; it is not clear to us that the majority of cases fit the 2-party full-synchronization pattern rather than some more complex multi-party or multi-stage scenario. We did not want to force all programs to pay for wiring the former assumption into all communications.

### Why are channels half-duplex (one-way)?

Similar to the reasoning about default-sync: it wires fewer assumptions into the implementation, that would have to be paid by all use-cases even if they actually require a more complex communication topology.

### Why are channels weak?

To simplify reasoning about resource-ownership and task-lifetime reasoning. This decision may be revisited in the future.

### Why are strings UTF-8 by default? Why not UCS2 or UCS4?

The `str` type is UTF-8 because we observe more text in the wild in this encoding -- particularly in network transmissions, which are endian-agnostic -- and we think it's best that the default treatment of I/O not involve having to recode codepoints in each direction.

This does mean that indexed access to a Unicode codepoint inside a `str` value is an O(n) operation. On the one hand, this is clearly undesirable; on the other hand, this problem is full of trade-offs and we'd like to point a few important qualifications:

* Scanning a `str` for ASCII-range codepoints can still be done safely octet-at-a-time, with each indexing operation pulling out a `u8` costing only O(1) and producing a value that can be cast and compared to an ASCII-range `char`. So if you're (say) line-breaking on `'\n'`, octet-based treatment still works. UTF8 was well-designed this way.
* Most "character oriented" operations on text only work under very restricted language assumptions sets such as "ASCII-range codepoints only". Outside ASCII-range, you tend to have to use a complex (non-constant-time) algorithm for determining linguistic-unit (glyph, word, paragraph) boundaries anyways. We recommend using an "honest" linguistically-aware, Unicode-approved algorithm.
* The `char` type is UCS4. If you honestly need to do a codepoint-at-a-time algorithm, it's trivial to write a `type wstr = vec[char]`, and unpack a `str` into it in a single pass, then work with the `wstr`. In other words: the fact that the language is not "decoding to UCS4 by default" shouldn't stop you from decoding (or re-encoding any other way) if you need to work with that encoding.

### Why is `log` an expression rather than library function?

We made this decision at a time when the syntax extension system was less mature. We plan to change `log` to be a macro (see #554).

### Why are strings, vectors etc. built-in types rather than (say) special kinds of trait/impl?

In each case there is one or more operator, literal constructor, overloaded use or integration with a built-in control structure that makes us think it would be awkward to phrase the type in terms of more-general type constructors. Same as, say, with numbers! But this is partly an aesthetic call, and we'd be willing to look at a worked-out proposal for eliminating or rephrasing these special cases.

### Can Rust code call C code?

Yes. Since C code typically expects a larger stack than Rust code does, the stack may grow before the call. The Rust domain owning the task that makes the call will block for the duration of the call, so if the call is likely to be long-lasting, you should consider putting the task in its own domain (thread or process).

### Can C code call Rust code?

Yes. The Rust code has to be exposed via an `extern` declaration, which makes it C-ABI compatible. Its address can then be taken and passed to C code. When C calls Rust back, the callback occurs in very restricted circumstances.

### How do Rust's task stacks work?

They start very small (a few hundred bytes) and expand dynamically by calling through special frames that allocate new stack segments. This is known as the "spaghetti stack" approach.

### What is the difference between a reference and a box pointer?

* A box pointer points into a reference-counted heap allocation.
* A reference points to the interior of a stack _or_ heap allocation, and formation or duplication of an alias does not entail reference counting.
* References can only be formed when the referent will provably outlive the reference.
* References can therefore only be declared in function signatures, as parameters.

### Why aren't function signatures inferred? Why only local slots?

* Mechanically, it simplifies the inference algorithm; inference only requires looking at one function at a time.
* The same simplification goes double for human readers. A reader does not need an IDE running an inference algorithm across an entire crate to be able to guess at a function's argument types; it's always explicit and nearby.
* Parameters in Rust can be passed by reference or by value. We can't automatically infer which one the programmer means.

### Will Rust implement automatic semicolon insertion, like in Go?

For simplicity, we do not plan to do so. Implementing automatic semicolon insertion for Rust would be tricky because the absence of a trailing semicolon means "return a value".