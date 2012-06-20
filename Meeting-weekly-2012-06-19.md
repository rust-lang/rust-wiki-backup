# Agenda:
- Repurposing last-use analysis for improving error messages rather than guiding optimizations (see https://github.com/mozilla/rust/issues/2633  )
- Case classes
  - patrick, do you have a proposal for syntax of these?
      (it would be terrible if it took N lines to write each "arm of the enum" like you have to in true-OO. or, said another way, i'm afraid of losing the ability to write enums as concisely as we can now.)
- class -> struct
- Out-of-line methods
- Coherence
- resolve status update/0.3
- suffix inference is ready to land
- future typestate directions    

# Last-use Analysis
- tjc: #2633 discusses repurposing last-use to solely 
- ndm: current scenario: liveness computes last-use, borrowck removes entries
- ndm: last-use analysis seems to be opaque to users
- graydon: we went too far down the path of implicit copies/moves
- graydon: I am ok with any work plan that involves phasing them out
- tjc: pcwalton's proposal was to use last-use just to inform the warning messages
- graydon: so we'd be copying but emitting warnings
- pcwalton: warning is only triggered but only for malloc/mutable state
- pcwalton: talked to graphics programmers who claim that 4x4 matrices should always be by-copy because it helps compiler 
- graydon: when it comes down to the ABI, a lot of the "by value" things are actually passed by reference anyhow.  So there many layers down below.  We don't need to tailor the language for that.
- ndm: goes together with unary move
- everyone who agrees raise hand? *all hands raised*

# Case Classes
- pcwalton: 
    - Case classes unifies enums and classes into one more general entity (see Scala)
    - Enum type becomes a base class, subclasses can be defined which must be within the same module
    - I call it "closed classes"
    - pattern matching is possible, Rust compiler statically know the size, models Rust enums precisely
    - three primary advantages:
        - lightweight sol'n to "datasort refinements" using single inheritance tree
        - variants are types
        - shared fields, which can be very important—particularly for DOM
            - workarounds are somewhat awkward
    - my thought: introduce case class ability but make enum classes syntactic sugar
    - reactions?
- sully: it seems like a good idea to me
- bblum: I think datasort refinements are good
- pauls: unifying seems good
- graydon: I am not totally opposed.  I am not sure how datasort refinements winds up working at a codegen level?  Is it complex to handle that?
- pcwalton: no, because of closed classes we can flatten things into one numeric range
- graydon: I haven't worked with that pattern but it seems useful
- graydon: I like the closed class with fixed size vs open one
- graydon: fits in with the types of indefinite size (open types, if available, would have indefinite size)
- graydon: apart from syntax of declaring entries, what about the ctor syntax?
- graydon: min-max classes proposes `C { a:1, b:2 }`, how does that fit with `P(1, 2)`?
- graydon: for small patterns (option, result, etc) anonymous fields are good
- pcwalton: I had planned to tie this in with newtype syntax
- pcwalton: if you write something like `class MyInt(int);`, more or less equivalent to `enum MyInt = int;` but we can use this for the pattern matching as well
- pcwalton: basically ergonomics is same as today
- graydon: I am not opposed, seems coherent, put together a complete proposal.  Sounds tidy.  In particular how you designate open vs closed classes.
- pcwalton: I had hoped to avoid open classes
- graydon: People will want it
- pcwalton: well if you're not opposed, we have thoughts on it, I can sketch it out
- graydon: we don't have to impl form get go, but maybe good to reserve syntax

# Class vs struct
- pcwalton: `class` is unsexy keyword nowadays but maybe `struct` has retro appeal
- pcwalton: but also our classes are more like structs, really, because they lack many of the OO features
- graydon: it's a very overloaded word, struct is less... ambitious
- graydon: I don't care
- ndm: also been proposed to make struct pub by default vs class priv by default
- graydon: a weird C++-ism in any case

# Out-of-line methods
- pcwalton: classes give rightward drift, but more serious problem is that with our menangerie of pointer types we want to be able to designate methods of type `@T` and others of type `&T` or `~T`
- pcwalton: also conceivably we might want to `move self`
- eholk: yes please
- ndm: e.g., unwrap() in dvec/option
- pcwalton: my current favorite thought is an explicit self parameter a la Python
- pcwalton: so you write `fn foo(self: @T) { }` to make a method, must be in the same module as the type `T`
- bblum: can you write it without `self`?
- pcwalton: no, just `fn() {}` would not be a method
- pauls: could be a bit of a pain if the name of the class is wrong
- ndm: my big concern is the repeating of bounded type parameters
- pcwalton: yeah we can toss around the syntax some
    - fn(T) foo() { ... }
    - fn T.foo() { ...}
- dherman: this is probably best done in another forum
- graydon: maybe the existing impl is superior
    - existing impl syntax is not so bad and the rightward drift isn't so bad, worth it if it saves repeating type parameters too often
- pcwalton: another interaction, "static methods" in impls
- pcwalton: this is something I've been wanting, generally to have "ifaces" that enable you to "build an instance of this type", like `fn read() -> T`

# Coherence
- pcwalton:
    - Coherence == guarantee of only one instance of an impl per type
    - Avoids hashtable problem
    - Also helps ergonomics: no need to import impls
    - Also makes resolve complex, impl names don't act like other names
    - I would like to make a lot of what we call "kinds" into interfaces (eg., send, drop)
    - but without coherence this doesn't make much sense, multiple ways to drop a type?
    - technique I came up with is to restrict the types you can define impls on:
        - you can define on any nominal type defined in your crate
        - and you can implement interfaces defined in your crate
    - (editor's commentary: what you cannot do:
        - implement an interface not defined in your crate on a type not defined in your crate)
    - avoids haskell's linking problem because there is no way to have two crates define an impl of the same interface on the same type
- graydon: simplicity is one of its virtues but I am uncomfortable with inability for people to add new instances of an interface
- pcwalton: well, you can so long as you own that type, you could wrap with newtype
- pcwalton: there is also some kind of hybrid combination with niko's original coherence proposal
- graydon: given a sufficiently nice newtype this seems tolerable
- pcwalton: this is basically what scala does, they have made the newtype syntax really cheap---free, actually, with implicits.  that is going too far for us, but the core idea works for them.
- graydon: yeah, it's not the end of the world not to have access to an impl, though it's "nice"
- graydon: would it be possible to add types cheaply that are subtypes?  
- ndm: can you clarify
- graydon: there is kind of a third case in between open/closed where you add new methods but do not change the representation
- ndm: sounds like C#'s static fns that you get to write with a.b() syntax
- graydon: current resolve rules seems too complex to publish, too complex to have more than one impl—perhaps too complex to even have one impl
- graydon: by far in favor of shipping a language with simpler rules that actually work

# Resolve Status Update
- pcwalton: About 3/4 of the test suite passes
- pcwalton: remaining major item: impls
- pcwalton: somewhat slower than it was but still 4-5x faster than it is today
- pcwalton: still true that we are planning to ship 0.3 when it is ready?
- tjc: think so?
- graydon: fits in with longer term plans to stabilize the language
- graydon: reaching the "ship or slip" pointer
- graydon: other thing was various "easy" syntax languages
- graydon: but we should not gate 0.3 on those
- graydon: e.g., getting rid of bind, changing "use"=>"link" and so forth
- sully: speaking of stage3 snapshots... how are the buildbots doing?
- graydon: IT folk are looking into it, linux1/linux2 are down, mac needs to be reimaged
- graydon: more VM hosts also available
- brson: can I get into those new machines yet?
- graydon: I think this is a problem where python script goes into infinite loop, not sure why

# Suffix inference
- lkuper: seems to be passing tests but for one failure
- lkuper: basically seems ok
- lkuper: a little bit concerned with some of the specific behavior
- lkuper: originally it was going to try and be smart about the width based on the value
- lkuper: but we ran into some difficulties, right now it just tries to infer a consistent type, and defaults to int if there is a lack of constraints
- lkuper: are we all happy with that?
- pcwalton: what are some of the tricky cases? I know that there was some question as to programmer's expectations... for example, `let x: uint = -1` might be reasonably expected to work.
- lkuper: we allow that today and the inference will also accept it
- bblum: it might also be sane to require an explicit cast (`-1 as uint`)
- graydon: but that seems slightly different, in that you can cast any numeric type, though I think it produces the same value in this case, but it might not do the same thing on a 32 bit platform with casting to u64
- bblum: that seems like a gotcha' in any case
- pauls: can we forbid that?
- graydon: how?  
- graydon: yeah, bit of a gotcha that `let x: u64 = -1` vs `let x = -1 as u64` vs `let x = -1u64` are all different on 32 bit platforms
- bblum: can you do something where you act differently for unconstrained variables inside an `as`? issue a warning?
- lkuper: we are not special casing at all, right now, on the context where the literal occurs
- ndm: you don't need the type checker, any literal inside `as`
- bblum: are there other cases where this can occur?
- graydon: integer indices make a difference?
- pcwalton: with suffix inference, can we change indices to be uint only?
- graydon: I am ok with trying to land it and we tweak the rules as we go
- graydon: not sending to ISO yet
- eholk: sounds strictly better than what we have
- graydon: lint passes seem like a reasonable solution
- ndm: good point, we can use lint passes also to detect constants that get resolved to unreasonable ranges (512 as i8 and so forth)

# Typestate
- graydon: How many people are in favor of keeping a strong typestate system?
*tjc raises hand*
- graydon: building things in boils down to making things easier to use
- graydon: idea of typestate was to make named constraints on data part of the "mood" of the language, but we're not seeing a lot of use right now
- graydon: current design seems to require too much manual checking, more automatic propagation would seem to be necessary
- graydon: we've been talking a lot about there being a const kind, as well as freeze and thaw, which we tend to need for concurrency
- graydon: now concurrency is a different topic...but it has a similar flow model...
- graydon: perhaps we can move named constraints to constructors where you switch over to const kind
- graydon: basically we eliminate the flow analysis and attach the checking of constraints to construction of instances of a particular type
- graydon: not clear to me that this will work 
- graydon: I see a spectrum, one hand there is a pull request to rip out tstate, on the other there is "Leave it as it is, try and enhance it so we get more static coverage from a single check"
- graydon: maybe there is a middle ground where we move the typestate checking to construction events
- graydon: arguably this is just a newtype'd constructor, where you just instantiate a newtype and check the constraints in the ctor
- ndm: I like the idea, basically instead of a check you use a ctor, and this moves to a pattern
- graydon: maybe this dovetails with the idea of "refined subtypes" that have same size
- eholk: it seems that typestate is a nice idae but in practice we haven't developed culture around using it.  If we can solve that it'd be cool but I also like the idea of using this in the type system.  Do we lose expressivty?
- tjc: not sure we know yet
- ndm: my experience has been that you can use these in each case
- graydon: it's a bit like phantom types, you are still pushing the burden back, for example a function that takes two vectors but instead takes a pair of two vectors
- ndm: I'm not sure that you'd get pushback if API were well written
- graydon: another way is to add syntax for pre and post condition that will just get run, basically another syntax for assert
- ndm: basically like eiffel?
- graydon: yes, but without the class invariants, which gets quite complex (enforcing visible states invariant is tricky--that is, allowing methods that do not require class invariant holds)
- eholk: can we ever use type state to remove safety checks?
- ndm:  yes in some simple cases but can be modeled with types too
- graydon: not hearing a lot of pushback that we must maintain current system I'm comfortable moving in more limited direction

# Other topics
- lkuper: 0.3 timeline?
- pcwalton: when resolve is done
- graydon: and we get buildbots back online, get a green tree, write up release notes, etc
