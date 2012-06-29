## Attending

Dave, Brian, Sully, Graydon, Lindsey, Eric, Ben, Paul, Elliott, Patrick

## State of vectors

  * **Graydon**: used to have one vector type, going to four (!) -- but hey we have four pointer types!
  * **Graydon**: two of them perform really well
  * **Graydon**: fixed-size, uniquely-owned, slices -- anything that is a consumer of a vector you want to write in terms of slices
  * **Graydon**: utility of this is that a slice is not necessarily in the heap or anything; just a pointer and a length; could be interior, part of a stack frame, sub-component of existing vector
  * **Graydon**: to make the transition, had to change existing vector syntax -- existing code means unique vector with [ ] syntax
  * **Graydon**: currently: /~ means unique, /@ boxed, /_ fixed size, /& slice -- these are horrible, everyone said they don't like the syntax, but we haven't settled on final syntax
  * **Graydon**: syntax still up in the air; but use the ugly syntax now and we'll bulk update later
  * **Graydon**: in compiler: evec/estr are for the new "(E)xciting!" data structures
  * **Graydon**: the code is confusing and ugly, and we're shipping with that, but 
  * **Lindsey**: what does the E really stand for?
  * **Patrick**: extended
  * **Sully**: vectors: there are wins we get for stack-allocated vectors, but not sure about strings
  * **Graydon**: b/c they're always immutable... long-term plan: constant expressions will be differentiated and will be compiled in read-only memory; that's currently already happening for strings
  * **Graydon**: we just don't yet have the logic for differentiating const from non-const vectors yet

## Resolve

  * **Patrick**: passing the test suite; not compiling compiler yet
  * **Sully**: can it do core or std?
  * **Patrick**: no
  * **Patrick**: std has name resolution problems; may just be relying on resolve1 bugs, which is easy to fix
  * **Patrick**: usually the problems are code relying on resolve1 bugs
  * **Graydon**: crashing compiler, or nice errors?
  * **Patrick**: nice errors
  * **Patrick**: for core, it's crashing compiler b/c of the way impl scopes work; some really brittle code in trans that relies on intricate details of how resolve1 works; I'll probably change that b/c it's bogus
  * **Patrick**: gonna take a little bit, shouldn't be too much longer

## 0.3 Release

  * **Graydon**: made release notes, please look it over and feel free to add entries
  * **Graydon**: small number of syntax changes we're cramming in at the last minute; if you're aware of syntax change that you think is pending that we're ready to make and that everyone's agreed on, there's a few days left so now's not a bad time to make those changes
  * **Graydon**: hopefully shipping the native -> extern change
  * **Graydon**: sigils for safe references, not sure we've reached consensus
  * **Graydon**: don't want too many more release w/ radical syntax changes
  * **Graydon**: sounds like we're not shipping this week
  * **Patrick**: might be this week, but it depends on how many bugs are lurking

## Traits

  * **Lindsey**: started reading original traits paper; presented as a way to get around problems of inheritance; in Rust we don't have inheritance
  * **Patrick**: that's changing; moving in a direction of some inheritance
  * **Dave**: well, traits are composing functionality regardless of whether you have inheritance
  * **Lindsey**: we have regular impls and iface-less impls; trait provides implementation but different from an impl b/c it doesn't have to be for a particular type; sort of like a type-less impl
  * **Graydon**: typically carry constraints on self
  * **Lindsey**: yes. standard thing for trait is required fields/methods and set of provided methods; occurred to me that the required block could be described as an interface; that makes a trait basically just an impl of an interface but without a type
  * **Sully**: to some extend we already support that; right now you can write `impl<a : i> for a` -- for any type that satisfies the interface. this only works sometimes
  * **Graydon**: I've never seen that. does that actually work?
  * **Sully**: I have some tests where that works
  * **Graydon**: the impl code is opaque to me, it's extremely clever code
  * **Patrick**: the semantics are really complicated. brian and I have been talking about that. with coherence I'd like to tame that a little bit. coherence makes the semantics quite a bit simpler
  * **Sully**: here's a gist that works today: https://gist.github.com/2996840
  * **Sully**: I tried a similar thing to get rid of the iter trait hack we have
  * **Lindsey**: promising since this says we do want traits
  * **Patrick**: I wonder if traits can be reformulated as syntactic sugar
  * **Lindsey**: that's what I was wondering
  * **Patrick**: with coherence you wouldn't have that name anymore, you'd just have THE implementation of to_str for A. hm, weird. I have to think of how that interacts with the coherence check
  * **Lindsey**: how is that different from an iface-less impl?
  * **Patrick**: it has an iface. the iface is the iface that contains exclaim. you say "this is the impl of exclaim given that the type that I'm implementing it on is a to_str". so it works fine with coherence. you have to define an iface, but traits need a require block anyway
  * **Patrick**: we could say we're done, we don't need traits, but that syntax is mind-bending. syntactic sugar seems like a possibility
  * **Lindsey**: I don't know how coherence works

## Coherence

  * **Patrick**: impls are currently named; you have to import the name to use them. people forget to import the name; name resolution gets really complex
  * **Dave**: I'd also point out it's more heavyweight
  * **Patrick**: if we want to formulate things like `drop` as ifaces, you really don't want multiple ways to drop a type. that's strange. you'd have to import an impl in order to satisfy the *implicit* call to drop that happens with an instance of a type goes out of scope. that'd be really weird
  * **Graydon**: no, the compiler could do that
  * **Dave**: if there's more than one implementation, the compiler doesn't know which one to choose
  * **Patrick**: I'm talking about user-defined drop
  * **Graydon**: oh you're talking about eliminating destructors except as an iface
  * **Patrick**: yes
  * **Patrick**: other problems: say you create a hash_map<str, str> using impl A of hash on strings, and I pass that hash table to other code and that code wants to look up things on the table, but if that code has impl B in scope for strings, it'll use the wrong hashing
  * **Patrick**: there are solutions but they mostly involve not using ifaces
  * **Patrick**: for those reasons (comprehensibility, simplicity of compiler, merging kinds and ifaces), coherence is worth doing
  * **Graydon**: what you mean by coherence is there being some limitation on the number of implementation of an interface
  * **Patrick**: at most one impl of an iface per type
  * **Patrick**: you ensure it by construction. Haskell's is brute force -- the linker throws up its hands if it encounters more than one. more elegant way to make it by construction so you can't do that: you can implement ifaces either in the crate that defines the type or the crate that defines the iface, with intra-crate compiler check that there's not more than one
  * **Patrick**: make sense?
  * **Graydon**: underlying difficulties there: places a bit of burden on iface developer that the crate where an iface is defined inside of has to define all of the interesting structural type variants at that point
  * **Patrick**: and built-in types
  * **Graydon**: yeah, the ones that don't count as nominal
  * **Graydon**: strikes me as a little bit awkward. leads to potentially hard-to-resolve case where someone forgets to do u64 or something, and client doesn't have access to the crate where that's defined
  * **Graydon**: concept of either extension interfaces or some sort of workaround
  * **Sully**: you can do a newtype wrapper
  * **Graydon**: sure, if it's possible to wrap in a nominal type and possible to have a subtyping relationship between them
  * **Dave**: or maybe it starts looking like a simplified version of earlier features like scoped extensions or named impls, just targeted to only this 20% case
  * **Sully**: how would this interact w/ the gist I pasted above?
  * **Patrick**: iface exclaim, then impl that iface; it would be the only implementation for any A that implements to_str
  * **Dave**: pretty much the same as Haskell, right?
  * **Sully**: not sure
  * **Patrick**: I think you just argued that interface extension subsumes traits
  * **Dave**: <blinks stupidly>
  * **Sully**: Niko and Graydon and I were discussing this
  * **Patrick**: for numbers you definitely want interface extension
  * **Dave**: I don't understand whether extension actually subsumes traits, in particular with multiple inheritance
  * **Sully**: traits have to be a DAG, don't they?
  * **Patrick**: no, they can have cycles
  * **Dave**: trait inheritance is structural, iface inheritance is nominal, so they're different
  * **Dave**: it's a good goal to try to minimize concepts, but it's not obvious to me that one strictly subsumes the other; and of course if we have a most general concept, it shouldn't be too powerful (e.g., coherence is about restricting usefully)
  * **Graydon**: I agree that we should minimize concepts; nominal types have a location in a module tree; we often wind up with duplicate code b/c people don't want to indent their code; think notationally of ways to associate an impl with a module and allow everything to be written flush-left with the module
  * **Dave**: that's how derek dreyer did his unification of typeclasses and modules, unifying around "the distinguished type exported by this module" convention from ML
  * **Graydon**: worth considering unifying around module-as-a-named-thing
  * **Patrick**: if you think of impls as kind of like a type module...
  * **Graydon**: our extension impls often have meaningless names
  * **Patrick**: so some goals: reduce rightward drift
  * **Sully**: just one level indentation, right?
  * **Graydon**: look at the vector code, all your methods suffer from it
  * **Graydon**: I think there's a reason C++ and Go implement out-of-line methods
  * **Patrick**: agree, which is why I originally wanted out-of-line methods. people didn't like them but maybe they're worth bringing back. main objection was duplicating type parameters
  * **Sully**: with Haskell typeclasses, you have to indent the core typeclass functions, but that winds up being not that many functions; you don't throw functionality into the typeclass; we do more because of the dot-notation
  * **Dave**: that's not going to change
  * **Sully**: I would like to orthogonalize ifaces and dot-notation
  * **Patrick**: I think coherence helps this
  * **Graydon**: we were careful about coherence the first time through, didn't want link-time resolution so we drove it off of names; don't remember how Marijn felt
  * **Patrick**: I think he was okay with link-time resolution; I think he didn't like the extra names
  * **Graydon**: let's check in with him

## Linked failure

  * **Ben**: task-local storage basically ready to go
  * **Brian**: safe or unsafe?
  * **Ben**: unsafe. found a way to make interface safe but would've required lots of hacks; there's documentation on how to use it right and what use cases will break it
  * **Graydon**: we'll build safe abstractions on top
  * **Ben**: linked failure: want to spawn a task linked to this task; call "task group" set of all tasks linked to each other; can spawn new tasks not in the same group with, dunno, spawn_parented
  * **Ben**: within task group, bidirectional; other policy: if parent dies, kill child (like unix processes)
  * **Ben**: currently we have it other way around; that's weird and I'm not convinced that's what we want
  * **Ben**: furthermore if somebody really wants it they can implement it on top of linked failure with non-linked parented spawning if they use a port for notification
  * **Brian**: so you don't think children should kill their parents?
  * **Dave**: <giggles>
  * **Brian**: that's the only use case we have implemented right now
  * **Ben**: why?
  * **Brian**: we have some use cases in current system: tests are children of test runner... actually not true
  * **Patrick**: having a hard time thinking of cases where you don't want child to die when parent dies
  * **Dave**: nohup is an example
  * **Graydon**: this is modelled after Erlang; they talk about how the system only becomes interesting once you have a reasonably large constellation of tasks
  * **Ben**: send me the link to the Erlang info
  * **Ben**: for terminology: spawn_linked, synonym for spawn; spawn_parented (child dies if parent dies); spawn_unlinked; can also specify a port
  * **Dave**: how do you keep the syntax light with optional arguments?
  * **Brian**: we created a bunch of spawn functions in the past; then converted to a builder interface to simplify, but that got awkward, so created a few variants for common cases
  * **Brian**: neither solutions turned out all that satisfactory
  * **Patrick**: could you do a jQuery kind of method-chaining trick to make it palatable?
`Task().unlnked().spawn()`
  * **Ben**: I like that
  * **Dave**: how well can we optimize that?
  * **Patrick**: perfectly fine
  * **Eric**: move-mode self?
  * **Patrick**: we can get there eventually
  * **Brian**: all this sounds pretty good to me
  * **Graydon**: sounds like a good direction; haven't merged TLS but in process?
  * **Ben**: should be today or tomorrow
