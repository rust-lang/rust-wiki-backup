## Agenda

- << by more than width of type is undefined, do we care? (pcwalton)
- per-binding-site "mut" (pcwalton)
- closure reform and dynamically-sized types
- precedence of "as" (pcwalton)
- proposal for effect system (bblum)

## Attending

pcwalton, jclements, brson, eatkinson, bblum, aaron todd, eston, niko, tim, pnkfelix, jld, sully

## Intern introductions

- hi, everyone

## << by more than target width undefined

- pcwalton: bz reported two years ago. do we care? I think 'no'
- dave: what does undefined mean?
- pcwalton: llvm can do whatever it wants
- dave: so llvm semantics
- niko: inherited from c
- dave: any reason we would let it happen
- brson: costs more
- pcwalton: needsa check every time
- sully: why?
- dherman: no dependant types
- sully: you can mask it
- pcwalton: adds another CPU inst
- dherman: do we know llvm will never crash? do we know their semantics?
- pcwalton: no
- nmatsakis: looked at this. it does all kinds of things
- dherman: if we can't guarantee that llvm won't crash... need to bound behavior
- dherman: only sensible option is to mask
- pcwalton: guarantees the value is undecided .. <missed>
- dherman: if, on any particular platform <missed>
- sully: what does java do?
- pcwalten: idk
- bblum: right shift also?
- pcwalton: i think. there's a bug on this. search for bz
- pcwalton: i'm fine with unspecified behavior as long as we know
- dherman: good to have a list of all places with unspecified behavior
- nmataskis: java does what you expect
- jld: java masks off bits. <some spec recitation>
- sully: how often to we shift by non-constant amounts? the cost of masking may be nbd
- sully: x86 already has pretty bad support for variable width shifts. can't shift by an arbitrary amount

[Shift amount has to be in %cl (low 8 bits of rCX), but that seems to be more of a register allocation issue to me. — jld]

- dherman: fully deterministic java semantics and not-fully deterministic but typesafe semantics are defensible
- dherman: i'm comofrtable if things are unspecificied while still safe

## Per-binding-site mut

- pcwalton: currently mut is permitted in ad-hoc places
- pcwalton: on args and after let, not before pattern bindings, which is odd because sometimes you'd like to copy into pattern bindings and mutate your local copy
- pcwalton: (not totally convinced about this)
- pcwalton: put `mut` before arbitrary binding, subsumes all existing cases. putting `mut` before bindings makes them mutable
- pcwalton: increases expressiveness because one 'let' has some mut some not, also in 'match'
- pcwalton: would mean removing 'let mut', having 'mut' distribute over sub-bindings
- jclements: makes things more consistent
- nmatsakis: sounds like a win
- felix: what about binding large structures with '@' (as-patterns)
- nmatsakis: don't know why not. if it's not by-ref there will be separate copies on both sides. it's ok for one to be mut and one non-mut
- bblum 'ref mut' and 'mut ref' mean different things?
- nmatskis: you could imagine it meaning something but better to not allow it
- pcwalton: grammar is 'mut IDENT'.

<mumbling>

- nmatsakis: the main thing is not allowing both because <suckage>
- jclements: currently `ref [mutablitily] IDENT`.
- pcwalton: <gesticulating at jclements screen>
- nmatsakis: Today it is like this:

    pat := [ref [mut]] ident
   
- nmatsakis: tomorrow it will be:

    pat := [mut | ref [mut]] ident

- brson: Can say 'let ref mut ..` today?
- nmatsakis: These two things are equivalent today (and would remain so):

    let ref mut x = y;
    let x = &mut y;
    // x: &mut Y

- jbclements: first creates mutable binding?
- nmatsakis: neither is a mutable binding
- bblum: would like this:

  let mut ref mut x = y;
  let mut x = &mut y;

- pcwalton: consistent but absurd
- jld: which mut is which?

<brian gets bored, stops transcrribing>

- someone: thinks `mut ref mut` is currently valid syntax
- someone else: suggests that it will ICE
- jld: tries it while everyone else is talking; it does.  (LLVM assertion, so there's probably unsoundness involved.)

## Dynamically sized types

- pcwalton: dynamically sized types first?
- bblum: dst influences closure
- pcwalton: what are bblum's objections?
- bblum: I have a branch that adds Sized bounds for many scenarios
- bblum: ran this on std, fixed 1200 errors by adding Sized bounds, collected stats
https://github.com/mozilla/rust/issues/6308
- bblum: 2/3 of polymorphic functions needed Sized. we thought it wouldn't be bad becasue many have Copy, and if you've got that then you can also have Sized
- nmatsakis: Copy implies Sized
- bblum: Most functions don't have any bounds at all
- bblum: Significantly more intrusive than copy bound
- bblum: Most popular reason for Sized is using T as function return type. 1/3 of errors
- bblum: argument in favor: less than 1/2 of files in std needed Sized. 50 needed. 84 not.
- bblum: number will get better outside of std. rustc has 1/4 the errors probably
- bblum: does benefit outweigh cost of Sized everywhere
- sully: what's the benefit of Sized?
- bblum: immediate benefit is that we eliminate the error situation where people with `&[]` can't borrow to `&T`
- sully: they would almost always get an error saying it isn't Sized
- nmatsakis: no. there are a lot of impls that would work, as long as they don't copy
- bblum: other advantage is that in the future we would be able to embed DST's inside data structures. Option<[T]> is DST
- sully: I came to the conclusion that DST are more trouble than they are worth. have we thought about the interface that would allow them in Option?
- bblum: what?
- sully: how does allocation work. currently have separate code paths
- nmatsakis: this is one of the things pcwalton has been thinking about and it enables moving GC into libs, etc.
- pcwalton: middle ground: could allow typedefs to have DST, and just typedefs
- nmatsakis: what does that accomplish?
- pcwalton: raw [T] or Trait can appear as typedef param but not in function sig. can have one typedef for custom smart pointer on vector, get nice syntax. must be expanded away by the time you use in a function

<confusion>

- nmatsakis: I sort of see what you mean.
- bblum: it just allows you to write types differently
- nmatsakis: it expresses types you wouldn't be able to write otherwise
- nmatsakis: imagine `Gc<T>`. inside there's a raw `*T` or `~T`. <rambling>
- pcwalton: yeah, this doesn't make sense
- pcwalton: basic objection to not doing DST is it reduces expressivity of custom smart pointers which I feel are important, and already used in servo, and likely to increase there
- nmatsakis: you can have custom smart pointers that have a layer of indirection. main objection is that Sized is intrusive. Adding Sized is most principled path, you could imagine Sized is the default and require T: Unsized
- dherman: is this something that can be deferred to Rust 2?
- pcwalton: not backward compat
- dherman: what about if everything is Sized by default
- bblum: has the 'pure' problem
- nmatsakis: other approaches, want to write them up. could imagine compiler being smarter about when it can be sure there's a Sized bound. e.g. [T] and take arg ~[T]. inside function <can't follow example>
- nmatsakis: would still write Sized on type declarations. trick would be to look at in-scope parameters to discover Sized-ness <unclear>
- bblum: how is this different from inferring the trait in general?
- brson: can we take this back to the drawing board?
- pcwalton: i'm hesitant to lose DSTs
- bblum: usually I sympathise with effort to reduce allocations. with this the difference is 2 vs. 1 allocation.
- nmatsakis: twice as many
- bblum: the difference between 0 and 1 is good, but I've never had a situation where <something>
- nmatsakis: there are other benefits as well, I would like &* to borrow vectors or objects. brings uniformity. lot of people stumble on this. may be worth paying for.
- pcwalton: since Unsized is an anti-trait, maybe we can make it not look like a trait. A keyword like 'dynamic' before type param to make it look not loke a trait bound
- sully: can put closures inside custom smart pointers? how communication with smart pointers would work?
- pcwalton: yes, we've discussed. it amounts to creating ways to borrow smart pointers.
- sully: I'm largely concerned with how this stuff will get constructed

## Closures

https://github.com/mozilla/rust/wiki/Proposal-for-closure-reform  
http://smallcultfollowing.com/babysteps/blog/2013/05/30/removing-procs/  
http://smallcultfollowing.com/babysteps/blog/2013/06/03/more-on-fns/  

- pcwalton: I'm fine with `fn` always stack allocated and no facility for adding thunks. brson had concerns
- dherman: that alone forces you to do an explicit move, equiv of capture clause. i understand explicit vs implicit tradeoffs, macros can mask sins
- dherman: the issue is you're saying 'a, b, c' twice and that's awful. have trouble getting past the pleasantness of zero mentions of captures
- jld: if you had a syntax extension that gives you the free variables of an expression would that give you equivalent of zero
- dherman: don't have macros that look like builtin syntax
- dherman: this is an ergonomics question, vs ability to reason about code
- dherman: concerned about forcing you to name captures
- pcwalton: it's a capture by move, something totally alien to other languages without affine types
- nmatsakis: the language enforcing it is alien, but not the idea
- pcwalton: it's hard to explain that if you used it in closure you can't refer to it outside
- pcwalton: it's implicit - because you used it you can't talk about it
- dherman: you had a version of thunk that worked by copying, not by move .... 
- nmatsakis: you could capture a reference, then end up with a thunk with by-ref as part of the type
- dherman: because you said `&thunk` we could have variations
- nmatsakis: not a variation, just falls out of the construction, a thunk is a struct parameterized by the types of the thing it captures, and one of those can be a borrowed pointer
- dherman: if you wanted to have a reference it would have to be bound to a variable so you would have to create extra temps by hand. can you have some syntax like `&expr` that automatically hoists for you
- nmatsakis: it's tricky because if you don't have a capture clause then it's an expr but what if ee.g. use variable in two inconsistent ways. if it's & you are trying to take a ref to your local copy, but if you have a capture clause you can

    thunk!(ref x => ...)
    thunk!(x.clone() => ...) // to capture a clone of an ARC
    converting to:
        do foo(&x) |x| { ... }
        do foo((x.clone()) |x| { ... }

- dherman: in your proposal there wasn't a capture clause, just something like a capture claause
- nmatsakis: list of variables to capture
- dherman: thought it was a param list
- nmatsakis: that's what it desurgars to
- dherman: the macro has a capture clause
- dherman: this feels like a very common case. we need to think of what the common cases are, and if they are not blessed in the central language, then we can't be asking users to use it
- felix: what about fmt!
- dherman: fmt! is fine. it's way different from building closures
- pcwalton: 
- brson: twiddle fns are used in many places, not just spawn
- dherman: I agree it should be in the library, but the language has been designed around this. if central idioms in the language ...
- dherman: fmt! is so familiar from other languages
- jclements: i agree with patrick. closures themselves have unusual syntax
- pcwalton: having to use a weird macro to spawn a task is a hard pill to swallow
- dherman: maybe you'll decide the ! isn't that important. my point is the ergonomics of this pattern needs to be smoothed out. this is a central pattern and it needs to feel natural.
- dherman: the complexity budget. your work on multiple pointer types is important because its the fattest part of the complexity budget, second is multiplicity of function types. huge complexity cost
- dherman: niko's doing important work, but some variations of his blog posts blow up the complexity budget which makes me nervous
- dherman: need to have holistic view of fn types and forms. need to be minimal as possible. important to have minimal function types, less important to have multiple expr forms
- dherman: important that whatever soln takes both seriously
- nmatsakis: I did a little slight of hand in that post. thunks are equiv of ~ fns but they are implicitly run once. they pass them in to body and you can move them out again. since you are saying that tilde functions are used everywhere. all I can see are run-once.
- brson: for most part unique closures are once fns, though there are a few places in the event loop where you want to reuse, timers, main-event loop, don't want to rebuild those every time around. Biggest use is creating your own variations of spawn and thus are almost once fns.
- nmatsakis: in cases where you want to reuse, if sufficiently rare, you could address with traits, or with extern fn(&mut T) and a "userdata argument" ~T
- pcwalton: non-once closures are kind of busted anyway since you can't clone upvars
- pcwalton: if you close over an arc your closure isn't copyable
- nmatsakis: we can have object types that have all these properties and more. then the only case that's not handled well is implicit by-ref, which is why I left `fn` closures
- pcwalton: the danger is that making things objects we'll become like java where you have to wrap your closures in objects, which everyone hates
- dherman: everyone hates the syntactic overhead, which we have macros for. in terms of mental model java's approach is good because it exposes the environment in the data structure
- dherman: i'm a big fan of the idea
- pcwalton: action items, Niko can post up his thought about DST.
- pcwalton: Read latest thunk proposal which seems to be heading in a direction that reduces complexity budget and accommodates imp't use cases from the scheduler
- dherman: A high-level point, I've been chatting with others, who have encouraged me to think about "ambient computing era" and whether Rust applies. Seems to me that the more that Rust becomes a seriously low-level language, which could plausibly target sensors etc, the better poised it is for this.
- pcwalton: besides C++11, nobody is really in the space that we are in
