# Agenda 01/21/2014

* default type parameters (acrichto) https://github.com/mozilla/rust/pull/11217
* I/O clone() vs split() vs select() (acrichto) https://github.com/mozilla/rust/issues/11165
* warn(unused_must_use) (acrichto) https://github.com/mozilla/rust/pull/11754
* optional type parameters (pcwalton)
* allocators (pcwalton)
* remove #\[foo="bar"\] (pcwalton)
* dynamic_lib in std, or "extra"? (pnkfelix) https://github.com/mozilla/rust/issues/8157

# Attending

- jack, larsberg, acrichto, nrc, pcwalton, brson, pnkfelix

# Status

* acrichto - rewrite the build system, mutexes again, pushing the queue through
* brson - roadmap
* pcwalton - Servo performance, remove @str

## Etherpad usage

- pcwalton: IT services says you can't use a stable etherpad with the same name over and over because they get worse with more edits over time. They recommended moving to google docs if you want scalability, etc.
- brson: ugh!

## friend of the tree

- brson: Jeffrey Olson has been contributing since Feb 2012. He did the original libuv integration, made our second attempt at implementing I/O, helped us port many parts of the C++ runtime to Rust, implemented file I/O in the green threading portion of our newest runtime and last week published an article about file I/O on the Safari Books Blog. Because of that, he's a friend of the tree!

## default type parameters

- acrichto: on the agenda for patrick!
- pcwalton: They are good because... allocator... << Vidyo troubles breaking up >>. They're good for allocators, and that's probably why we need them. Eddyb's implementation was good, so I propose we take them.
- acrichto: semantics?
- pcwalton: Unsupplied, it defaults to the type given (have to be at the end). `type Hashmap<K, V, A=Heap>` and if you don't provide an allocator/argument for A, you get Heap.
- pnkfelix: If we add bounds, the default would have to satisfy it, right?
- acrichto: Don't they *always* have to have a bound?
- pcwalton: Could have a vector or something that defaults to int. Don't know why it would be useful.
- acrichto: Syntax `A:Allocator=Heap`?
- pcwalton: Yes.
- pnkfelix: Alternative would be to use type definitions to create new names that feed in the default values. Only reason that's not palatable because you can't call methods through type expresions using type aliases, right?
- pcwalton: That solution is also more verbose.
- pnkfelix: But it's more expressive, too, because you could have differently named type aliases.k..
- acrichto: Documentation would be on HashMapAllocation instead of HashMap.
- pcwalton: Also have to have separate names, like AHashMap, etc.
- acrichto: Error messages?
- pcwalton: Should omit the default type parameter by default.
- brson: Any other use cases?
- pnkfelix: Hash function for a HashMap?
- pcwalton: Could imagine it... especially once we do the closure and function conversion. Also SortBy. When we have unboxed closures as type parameters, could imagine having a default SortBy where the default calls the LT operator.
- pnkfelix: Feature-guard this?
- brson: Probably should.
- acrichto: It'll be touch, because not just the definition, but also the usage.
- pcwalton: Depends on how thorough we want to be.
- acrichto: Seems like not a core thing, but the icing on top. I like having it gated.
- brson: If the only use case is allocators, we should do an experiment with those allocators. 
- pnkfelix: Probably not the only use case.
- brson: Niko was reluctant, but I think pcwalton convinced him via all the design converstaions.
- acrichto: So, you'd have to opt into the feature gate to use a non-default value of the default one?
- pnkfelix: If you feed all the parameters, you should be safe...
- pcwalton: But then everybody using HashMap has to use the gate!
- brson: Prefer to pretend the default paramter does not exist, and gate in to apply something. We can remove the feature. Are these left-to-right, so apply the left side one before the right side ones? Sounds OK. Maybe suggest to eddyb that he keep working on this and allocators.
- pnkfelix: Does he care about allocators?
- pcwalton: He's said so in the past.

## allocators

- pcwalton: What did we decide in the past on allocators? It seemed to work...
- brson: I can try to dig them up.
- pcwalton: Wish niko was here. He has a nice example that doesn't use HKT, etc. Maybe we shouldn't talk about this until he's around.
- brson: He will definitely have an opinion; we should skip it.
- pcwalton: Idea was an allocator provides a unique ownership semantics and you can build other things on top of that (Refcounting, etc.). Separates ownership semantics from allocation, which was the important thing. I don't remember the details.
- brson: I will forward around the notes.

## removing dynamic lib to extra

- pnkfelix: syntax extensions require this. Can libstd depend on something in extra?
- brson: Yes.
- acrichto: I think the crate map uses dynamic lookup... maybe we can just use the C lib functions directly for Windows.

## warning about unused results

- acrichto: Were two modes: warn unused result - semicolon statement without a unit return type gets warning. (off by default) SEcond was if the semicolon was used on a must-used type (on by default). I think this is our way to remove conditions from I/O. All I/O will be resturning results of Something, IOError. Since it's must-used, you can't do "let _ = foo.write();". Other ideas is to have a macro so that if it's the OK value, you are fine, otherwise you keyword return the Error expression. Almost preserves default fail semantics - keeps the requirement that someone *must* look at it.
- brson: Must_use on types instead of methods?
- acrichto: Started with default warning on everything, but too many false positives (30-40%). That's bad enough not to have it on by default.
- brson: All Results should be used, though?
- pcwalton: Methods is imprecise because of higher-order functions. On types is better.
- brson: Interesting policy decision...
- acrichto: Look at return type on the semi of an expression, you could look up the method, but this is the easiest thing that we have. Just look at the type.
- pcwalton: Worried about on methods imprecision on higher-order functions. Types are also imprecise because of generics, but it's much less imprecise than functions.
- brson: Any other types you added this to?
- acrichto: Just Result
- brson: So if you want to must_use an int, you have to newtype it?
- acrichto: Yes. On warn_by everything, there were tons on C functions.
- jack: Use must_use on both types and methods? Seems like valid cases for both.
- pnkfelix: c_int woudl be good.
- acrichto: Maybe on typedefs...
- pnkfelix: If you use `let _ = ...' form, do you still get a warning, or is that sufficient?
- acrichto: It shuts off the warning. Only works for statements of the form `expr;`
- brson: Unrelated on this PR, the try macro rubs me the wrong way. Already use try to mean something else. Just the name.
- acrichto: I didn't implement that; it was just my idea of what to do about errors.
- pnkfelix: Still think dynamic variables will be necessary. I can understand that's what conditions act like now. I like people handling errors locally, but that's not always going to be the right thing.
- acrichto: dynamic to say "fail-by-default?"
-  pnkfelix: e.g. in strings say "replace failed encoding character with {FAilure, replacement-char, etc.}". We can try this approach for now, first.
- acrichto: That seems odd to me. The signature would still return Result, it's just that there's an implicit contract because there will never be an Error. Still be warned about it, so you'd have to use `let _ = ...`.
- brson: You're talking about I/O specifically. dynamics not best for I/O. But for string encoding, they work pretty well. Everybody's pretty much in favor of this?
- pcwalton: I like this. 
- pnkfelix: I'd like to try it out first... but go ahead and push it.
- brson: This will make the Option vs. Result distinction bigger.
- acrichto: Basically, you're saying if you want people to look at the failure case if you return Result. I'll brainstorm on a better name for the try macro
- brson: Maybe a monadic do as well...

## attributes

- pcwalton: Talking about removing the metaname value. It's kind of random if you use that or a list with one element. The Inline uses one; langitems the other. Most languages don't seem to have this, so I'm proposing removing it. Also makes it easier to get rid of the brackets.
- brson: Allows reducing syntax. Problem: Name/value items and list items are different. ident=literal, but with list you have ident(metaitem). Can't say ident(literal), so there's no replacement. Might have to reforumulate the list type.
- pcwalton: Maybe just change the syntax instead of an equals to parens. That's what C# does.
- brson: If the list type can be either an ident or literal, that would be fine.
- pcwalton: Multiple literals?
- brson: I'd suggest it, but I don't know. Then you have to make a decision on a new attribute if it takes an ident or a string, which is pretty arbitrary.
- pcwalton: Idents unless you need spaces in there. So langitems are wrong right now.
- brson: Yes, probably.
- acrichto: Doc is the only on ethat uses spaces.
- pnkfelix: Maybe just make ident(literal). Is that too confusing for people? That the similar syntax has different things in the grammar?
- pcwalton: Only difference is if you allow multiple literals....
- pnkfelix: I was thinking of the nested case. Anyway, I don't mind it so much.
- brson: What's the benfit?
- pnkfelix: Main thing is that I don't see the use for multiple literals in a list. If I saw one that wasn't just abuse, I'd be more in favor of it.
- brson: A list of possible search directories?
- acrichto: Yes, right now we use that for path if your module is in a non-standard location. Big use of name=value is where it's pathname=name("pathname").
- brson: Do conservative for now and loosen later?
- acrichto: Where just one literal?
- pnkfelix: Every literal has a key name it's attached to in the current system... don't know if that's useful or not.
- brson: Can still have multiple attributes that duplicate the keys. I don't know if we use it as a key,value anywhere. This sounds like a good solution and probably immediately solves our problem of what to do with attribute syntax. Still not sure on # or @. But we can at least get rid of brackes `#!item(blah);`. Everybody good?
- pnkfelix: #^?
- acrichto: Outer was #!, inner was #^? Might be a difference sketch. I think we should remove name=value to free up syntax, though. 
- brson: No issue. Can you open one, patrick? To remove the name=value part. opened: https://github.com/mozilla/rust/issues/11886
- pnkfelix: Inner attribute syntax: https://github.com/mozilla/rust/issues/2569
- brson: bikeshed?
- pnkfelix: no.

## I/O

- acrichto: Can't simultaneously read/write a TCP stream. Multiple solutions: 1) select. Problem is that it's a very nebulous concept with complicated designs. Possibly not the best... once you have it, everybody wants to select over EVERYTHING. Two other options: 2) clone, 3) split. Clone lets you copy the stream, and this one returns Option<T> instead of just a T. Natural with the language. Downside is doesn't make sense to simultaneously read in two threads. OS will work, but it's racy. Also, channels have a reader and writer halves that are distinct. So why not do that with streams? So #3 splits the reader and writer halves, consuming the value. The type system preserves them. It's kind of annoying because you need TCPStream, TCPReader, TCPWriter. Final idea 4) is a tuple of a TCPReader and TCPWriter, then can define TCPStream over the tuple of the two. Cute! So you'd never see the tuple unless you destructure it. Then you have the bare minimum types, at least. Those are the general ideas - what do people think?
- pcwalton: I was uncomfortable with clone before, but better now. Sure, you can be racy with it, but can be racy with split, writing to a database - there are no issues with memory safety, so it's fine.
- brson: Leaving clone()'s semantics with multiple reads up to the OS makes me feel bad.
- acrichto: UV has a lock that sequentializes it.
- brson: UV will block on multiple reads?
- acrichto: We have to manually do it. UV corrupts itself, so we block all pending read requests.
- brson: You have the semantics in mine for all these cases and will define them?
- acrichto: For green. For native, you just get the OS. I could add a mutex around it so you only have max 1 active reader, active writer.
- brson: My inclination is towards split because of those issues.
- acrichto: method, or the tuple thing?
- brson: Encapsulate it so we have more options later is my favorite. If we expose them directly, then we'd have to always allocate a Reader and Writer. I'd be worried we would do that, want to reduce those allocations one day, and be stuck. Can we implement select later even if we do that?
- acrichto: No. Just one more thing to select over.
- brson: Clone seems easiest.
- acrichto: Clone was straightforward to implement, but it might not be what you expect for simultaneous read/writes.
- brson: Feedback on IRC / extended?
- acrichto: A couple in IRC said clone seems like a bad idea nad liked split. Could probably get more feedback on a PR.
- pnkfelix: Don't see discussion of clone on the issue...
- acrichto: I'll start a mailing list thread to get some more opinions on it.
- brson: What do you prefer, alex?
- acrichto: I kinda prefer clone. You do bring up a good point about relying on OS-specific things.
- jack: Maybe clone and if you call read or write, it disables the other? So it'd fail?
- pcwalton: That would be surprising.
- acrichto: Both could still call read. Unless you have shared that state across all clones...
- brson: As long as you have tests, it's OK to rely on the underlying system. If not, special cases on certain platforms to make them do the same thing. For progress, let's just do clone and see what happens.
- acrichto: I will open the PR.

## visit_glue

- acrichto: visit_glue is generated on every crate?
- pcwalton: Yes. Some attempts to try to avoid it, but it gets made anyway.
- acrichto: Uses codegen to do :?. Huonw added a -c to make it a no-op. He got some improvements in terms of file size and compile-time memory usage (like 5%). 
- pnkfelix: Crazy for thinking it should fail instead of no-op?
- pcwalton: Supposed to be for debugging.
- acrichto: No change to libraries. This is a problem: standard library should not have visit_glue anywhere. Consequence of this is that it generates float visiting, which means we rely on libm.
- larsberg: in servo we use :? in a lot of places. we'dhave to have this on all the time.
- acrichto: You could derive the empty braces. It is really nice to just be able to log any type in the world...
- pcwalton: the visit_glue is like deriving but uses a totally different path that generates code that never gets used. Deriving is the mechanism we have for things like that and it's very mature. The reflection is barely maintained.
- brson: FEels like reflection isn't pulling its weight. But reflection lets you print any time. With deriving, if it's in somebody else's crate, you can't show it.
- pcwalton: unless you newtype it. You can then impl Show directly.
- pnkfelix: THat's painful!
- jack: But what about private fields? How do you get to them?
- acrichto: We'd inspect it as well as we could...
- brson: If we're getting rid of reflection, I'm in favor of making this change.
- acrichto: I'd like a solution to opt out of it, because reflection is so useful for :? and is greatly undervalued. Codegen is a serious problem.
- pcwalton: Have we tried the deriving solution? Not a problem in OCaml... might be fine. 
- brson: WHy is genearation of visitors so imprecise?
- pcwalton: Existentials. If you wrap it in the trait, you don't know if it will be visited. I think we'd need a Show type bound on them, maybe. Because of monomorph, we do know if it will be visited or not. We could work on improving reflection... but I don't really want to touch that code. 
- brson: I don't think anybody is happy with visit_glue anyway.
- pcwalton: It's old, crufty, and unmaintained. Don't want ot hack on it.
- brson: If it's just in :?, we can just use the bad implementation for a long time.
- pcwalton: And pay codegen/compile-time costs everywhere? Further, it really hurts the changes we want to do for DST, vector representation, etc. Have to go redo the visitor glue every time, and it's a giant pain. This stuff has maintenance costs.
- pnkfelix: My vote is for having :? opt-in where if you not calling user code to print structures.
- pcwalton: Who will maintain it? 
- pnkfelix: I can't volunteer, since I haven't looked at ti. 
- larsberg: Won't you need this stuff (reflection) for a GC? You need that for leaks and debugging.
- pnkfelix: Was assuming we'd have type metadata I can look at for that; I don't know how much overlap there is.
- brson: Keep until later?
- pcwalton: Remove it. Or add deriving SHow and remove it.
- acrichto: WIth this PR which adds a -z option to not emit visit_glue, what should we do?
- brson: Is it just a stopgap or did they need it for something? Seems like a partial solution. I think we should do a full solution.
- pcwalton: I'd like to add deriving Show and then remove visit_glue completely. The vector representation changes will require a huge redo of the reflection APIs. It would almost have to be rewritten anyway.
- brson: Out of time. We'll come back to this. Thanks everybody.
