## Congrats on 0.1!

- http://news.ycombinator.com/item?id=3501980
- http://www.reddit.com/r/programming/comments/opgxd/mozilla_and_the_rust_community_release_rust_01_a/

## Monomorphizing, cross-crate inlining

- P: How best to proceed? Prob. start ugly then make it nice
- P: Having ability to "#include" (in the compiler) crates, basically "use" finds .rc files as well
- P: Ugly in long term, would probably prefer to separate out dynamic and static part
- G: No, no, no! Doesn't feel like it fixes the problem. Not a real sol'n.
- P: Ideal would be to write some repr into crate metadata.
- P: One thing you can also do is write source into crate metadata.
- G: Better in a way.  
- P: Would be recompiling libraries for every crate, but only (a) once per crate and (b) the static, generic parts like vec, etc. Mostly what's in core.
- G: Concerned that you will wind up with compilation artifacts
- Jesse: seems like that gives undue privilege to the built-in vec
- P: Important question: do we all agree it's a good idea? Is there an alternative for perf reasons? Want to get rid of hairy type desc logic in trans/shape
- G: Reflection?
- P: An important consideration.  C# gets around it by having a JIT.  We don't have one.  
Moving to IRC. Vidyo non-functional.

IRC Logs:

<pre>
[5:30pm] nmatsakis: all accounted for?
[5:30pm] brson: yes
[5:30pm] brson: let's go!
[5:31pm] nmatsakis: first, Jesse, to address what you said about giving undue priv. to built-in vec... I think what pcwalton meant is that any type or function could be monomorphized, but in practice it would mostly be the stuff in core
[5:31pm] graydon: hilarity
[5:31pm] pcwalton: yeah, basically we'd be introducing a sort of "STL"
[5:32pm] pcwalton: but it'd only contain stuff that actually has to be statically linked -- i.e. generic functions
[5:32pm] pcwalton: the rest will be dynamically linked
[5:32pm] pcwalton: I mean, I guess when we ship a browser we will probably want to statically link everything
[5:32pm] graydon: possibly
[5:32pm] pcwalton: but for development you don't want to recompile the whole stdlib
[5:33pm] graydon: yeah. most of the time, I think, you don't.
[5:33pm] Jesse: will monomorphizing and inlining be up to the optimizer? so it could depend on how hot the code is?
[5:33pm] nmatsakis: monomorphizing would not.
[5:33pm] nmatsakis: inlining would be.
[5:33pm] graydon: I'm kinda concerned that all interface-bound generics will wind up floating out to this monomorphizing layer
[5:34pm] nmatsakis: what does that mean precisely?
[5:34pm] pcwalton: Jesse: monomorphizing will always happen, but LLVM is free to make decisions about inlining (although it'd be nice to expose LLVM's always_inline, inlinehint, and noinline attributes as Rust attributes)
[5:34pm] fzzzy joined the chat room.
[5:34pm] nmatsakis: I do think that any function with an interface-bound generic will need to be monomorphized to be invoked.  Ah, and you're saying that this would not work so well with reflection or what have you?
[5:34pm] nmatsakis: because you wouldn't be able to produce a "generic" variant?
[5:34pm] graydon: I mean .. if we're encouraging people to write code more and more in foo<T : iface> form
[5:35pm] graydon: then more and more of the "normal" code people produce will actually be deferred-compilation this way
[5:35pm] graydon: making general compile times go up
[5:35pm] graydon: I want to be perfectly clear about something
[5:35pm] graydon: fast compile times are *really important*
[5:35pm] graydon: the reason people leave C++ is because of the build-cycle time as much as the complexity
[5:36pm] graydon: they lose flow. productivity tanks.
[5:36pm] nmatsakis: something we have to work on in general.
[5:36pm] graydon: I do not want to get into a situation where the normal way to normally approach rust development tasks is "compile time is O(sizeof(all-used-libraries))
[5:36pm] graydon: really, really don't want that
[5:36pm] graydon: that makes us C++ and we are back in the "static languages are too slow to use" camp
[5:36pm] pcwalton: agreed, we need to continue to support dynamic linking
[5:37pm] nmatsakis: It's not clear to me that we are encouraging people to write foo<T:iface> unless T is really unknown. 
[5:37pm] nmatsakis: However.
[5:37pm] graydon: you're effectively turning on mandatory whole-program optimization all the time if that becomes the default mode
[5:37pm] graydon: it's fine for JITs, not fine for AOTs
[5:37pm] pcwalton: hmm, does the "boxed exception" cover most of the <T:iface> cases?
[5:38pm] nmatsakis: *However, maybe we can re-use "compatible" implementations to some extent anyhow...
[5:38pm] nmatsakis: (Sorry, can't find a way to phrase that thought...)
[5:38pm] pcwalton: i.e. are most of the Ts getting passed by reference and not copied?
[5:38pm] nmatsakis: pcwalton: I should hope so...
[5:38pm] pcwalton: because there's no problem if they're getting passed by reference
[5:38pm] pcwalton: that's basically just void *
[5:38pm] nmatsakis: well, we'd still have to pass in the dictionary
[5:38pm] pcwalton: that's fine
[5:39pm] pcwalton: I mean, monomorphizing away the dictionary should be an option
[5:39pm] pcwalton: but it doesn't have to be mandatory
[5:39pm] graydon: ok
[5:39pm] graydon: that's what I was worried about
[5:39pm] pcwalton: it's basically the CRTP vs. vtable inheritance.
[5:39pm] pcwalton: sometimes you want one, sometimes you want the other
[5:40pm] nmatsakis: how would we make it an option?
[5:40pm] pcwalton: a special keyword, presumably
[5:40pm] graydon: if most iface-bound generics (i.e. "our most powerful and pleasant system for writing library code") are pass-by-ref-and-vtbl and we don't monomorphize them by default (absent, say, some "inline" keyword or such) then I think this is not as likely to blow up
[5:40pm] nmatsakis: or is that "in the future"
[5:40pm] graydon: I just want to be really careful about it
[5:40pm] pcwalton: we already have an "inline" keyword
[5:40pm] pcwalton: that's not used for anything at the moment
[5:40pm] graydon: casually walking into super-slow-compiles is something we must not do
[5:41pm] nmatsakis: it would be nice if we can define a "default" compilation mode that covers most cases
[5:41pm] pcwalton: another thing we should do in the future is to avoid compiling functions that aren't used
[5:41pm] nmatsakis: clearly, if we are going to suck in other crates, we'll *have* to do that...
[5:41pm] pcwalton: i.e. determine which library functions are actually called, and don't even parse the others
[5:41pm] graydon: yeah. that's also part of why I was real sketchy about "parsing .rc files based on 'use'"
[5:42pm] pcwalton: yeah, probably better to just read in crate metadata from the start
[5:42pm] nmatsakis: not parsing them would be easier if we stuck the (perhaps pretty printed) source into the metadata
[5:42pm] pcwalton: read in a serialized ast from the crate metadata from the start
[5:42pm] nmatsakis: (or serialized into EBML)
[5:42pm] graydon: it's much more likely that we'll *start* with the lazy, only-what-gets-used mode if we drive it from a later phase in the compiler
[5:42pm] anant left the chat room. (Quit: anant)
[5:42pm] graydon: yes
[5:43pm] nmatsakis: I am a bit concerned about how the "void*" case is going to work, particularly around copies.
[5:43pm] pcwalton: you just can't copy it
[5:43pm] nmatsakis: perhaps we'll have to pass in type descriptor anyhow to get take glue and so forth
[5:43pm] nmatsakis: ok, so if you declare <T:copy> then you're going to end up monomorphizing more than if you do <T>
[5:43pm] pcwalton: yeah
[5:44pm] pcwalton: but T:copy is likely mostly for stuff that you would want to monomorphize anyway
[5:44pm] pcwalton: (vector functions, etc)
[5:44pm] nmatsakis: it would be interesting to look at the places where we do <T:copy> and see to what extent we can eliminate those
[5:44pm] nmatsakis: but yes, the biggest case seems to be stuff like vec
[5:45pm] nmatsakis: I mean, we *could* handle copy of a ptr too if you treat it like any other interface
[5:45pm] pcwalton: but where do you copy it to?
[5:45pm] pcwalton: caller generally has to allocate
[5:45pm] pcwalton: and then you get into tydescs and GEP_tup_like again
[5:45pm] nmatsakis: the copy function allocates a new location.  if the type is @T this is task-local
[5:45pm] pcwalton: oh, ok
[5:45pm] nmatsakis: if the type is ~T this is a unique ptr
[5:45pm] nmatsakis: actually @T would just return the same ptr
[5:45pm] pcwalton: a limited form of copy that copies into a specific location
[5:45pm] ircloggr|beta left the chat room. (Client exited)
[5:45pm] pcwalton: or a specific type of location
[5:46pm] nmatsakis: right, your T would have to be a ptr type (so @T, ~T, *T)
[5:46pm] ircloggr|beta joined the chat room.
[5:48pm] pcwalton: anyway, this sounds plausible -- need a proposal for it
[5:48pm] anant joined the chat room.
[5:49pm] froystig left the chat room. (Quit: Computer has gone to sleep)
[5:49pm] pcwalton: aside from monomorphizing: I wanted to bring up semicolons
[5:49pm] nmatsakis: what it boils down to is that, for every generic fn, we could generate a default version that handles some set of conditions.  if you call it outside those conditions, we would have to monomorphize at that point.  if you are setup to dynamically link at the point of that call, then we would generate an error.
[5:50pm] nmatsakis: just to make sure we're all on same page...
[5:50pm] pcwalton: a bunch of people have complained about it and I talked with nmatsakis yesterday -- I think we could adopt ocaml's solution of ignoring the trailing semicolon if we started forbidding/making warnings out of unused function results
[5:50pm] froystig joined the chat room.
[5:50pm] kib2 left the chat room. (Ping timeout)
[5:50pm] froystig left the chat room. (Quit: froystig)
[5:50pm] froystig joined the chat room.
[5:51pm] Jesse: what if we monomorphize for just a few cases, where it's a significant win (e.g. T is word-sized and copying is trivial), and use the generic version for the cases that are expensive anyway?
[5:51pm] nmatsakis: Jesse: the problem is that monomorphizing buys you nothing when it's word-sized and copying is trivial
[5:51pm] graydon: ??
[5:51pm] nmatsakis: I mean, if you do the same operations
[5:51pm] graydon: I thought the opposite is true
[5:51pm] nmatsakis: sorry
[5:51pm] nmatsakis: you're right, I mean it buys you something over using a type descriptor
[5:52pm] nmatsakis: but it doesn't pay to specialize to each ptr type independently
[5:52pm] nmatsakis: when you do the same operations for all of them
[5:52pm] nmatsakis: that's what I meant
[5:52pm] froystig left the chat room. (Ping timeout)
[5:52pm] nmatsakis: but I think that if we do all the work to monomorphize and STILL have GEP_tup_like, we failed.
[5:52pm] nmatsakis: my main concern is that we'll be fixing bugs in that code till the day we die.
[5:52pm] pcwalton: yeah, that's one of my main concerns too
[5:52pm] ErikRose is now known as Erik|wfh.
[5:52pm] anant left the chat room. (Quit: anant)
[5:53pm] pcwalton: GEP_tup_like, GEP_tag, and the shape code involving generics are not only really hard to get right, they're really slow
[5:54pm] pcwalton: very few languages have done intensional type analysis like this
[5:54pm] pcwalton: (I actually don't know of any)
[5:54pm] pcwalton: the choices are generally uniform representation or monomorphizing (I'm counting C# in the latter category, it's just late monomorphization with a JIT)
[5:54pm] oconnor0 joined the chat room.
[5:55pm] Jesse: intensional?
[5:55pm] graydon: I agree
[5:55pm] graydon: I've ... "satisfied my curiosity" about whether it's the sweet spot
[5:55pm] pcwalton: Jesse: yeah, I read a few academic notes that called what we're doing "intensional type analysis"
[5:55pm] graydon: it's not an impossible sweet spot, but it's not *plainly* sweeter than the alternatives
[5:55pm] graydon: the compiler has to be bloody smart, and it's pretty fragile and performance-spooky
[5:55pm] Jesse: would it help to make a "T is a tag" version of the function without making a separate version for each tag?
[5:56pm] graydon: I'm ok trying other strategies too
[5:56pm] pcwalton: Jesse: not really, tags have varying alignment restrictions
[5:56pm] graydon: I just want "don't make compile time shoot up" to be a ... dominant consideration
[5:56pm] pcwalton: agreed, we need to tread cautiously here
[5:56pm] nmatsakis: I think that's fair.
[5:56pm] shoenig joined the chat room.
[5:56pm] nmatsakis: you're absolutely correct that it's important.
[5:56pm] graydon: I know building-servo-to-perform-well is a strong consideration. but servo-hackers-getting-frustrated-with-half-hour-compiles is equally valid.
[5:56pm] mccr8 joined the chat room.
[5:57pm] graydon: I'm already pretty pissed off about how much we serialize around LLVM
[5:57pm] anant joined the chat room.
[5:57pm] nmatsakis: I wonder if we would be able to use some of this infrastructure to cache results from previous compilations.
[5:57pm] pcwalton: you mean LLVM's slow performance?
[5:57pm] nmatsakis: and if we did that whether it'd buy us anything.
[5:57pm] graydon: no, I mean, even if we parallelize our FE and ME passes a lot, it feels like everything grinds into a bottleneck at LLVM
[5:57pm] pcwalton: yeah :(
[5:57pm] brson: we could parallelize trans couldn't we?
[5:58pm] nmatsakis: graydon: precisely why I'm dubious that caching previous results would buy us much...
[5:58pm] brson: llvm can be threadsafe can't it?
[5:58pm] pcwalton: I wonder if LLVM could implement a parallel FastISel
[5:58pm] pcwalton: since FastISel is strictly instruction-at-a-time that seems plausible to me
[5:58pm] nmatsakis: brson: do you have a sense for how much time is spent in trans vs LLVM itself?
[5:58pm] nmatsakis: I guess --time-passes would tell us...
[5:58pm] pcwalton: nmatsakis: build with TIME_PASSES :)
[5:58pm] pcwalton: I always build with it
[5:59pm] pcwalton: anyway, what do people think about semicolons?
[5:59pm] anant left the chat room. (Quit: anant)
[5:59pm] nmatsakis: so, I don't mind the semicolon rule as it is today.  I do however think that warning about unused function results can be very useful.
[5:59pm] nmatsakis: though it may take C programmers by surprise.
[6:00pm] nmatsakis: (I also wouldn't mind changing the rule to allow trailing semicolons)
[6:00pm] brson: I think if you have a warning about ignoring results then you need an attribute to mark function results as ignorable
[6:00pm] brendan left the chat room. (Quit: brendan)
[6:00pm] graydon: if we do "threadsafe" LLVM, does it actually ... uh ... go faster to hammer on it in parallel? I mean, how much of what it does is just build up an IR and then process that IR in a serial batch?
[6:00pm] nmatsakis: brson: why not just declare a function with a unit return type that calls the other function?
[6:00pm] pcwalton: graydon: I think there's not much processing involved in the FastISel
[6:00pm] brson: graydon: I don't know. I was thinking you could run the entire LLVM pipeline in parallel then use the linker
[6:01pm] pcwalton: it's pretty much just build IR, build assembly instruction-at-a-time and function-at-a-time
[6:01pm] pcwalton: seems parallelizable to me at a high level but I'm ignorant as to the details
[6:01pm] pcwalton: and the "build IR" phase is trans
[6:01pm] pcwalton: so as long as we can build functions in parallel
[6:02pm] nmatsakis: pcwalton: you mean with optimization turned off I guess
[6:02pm] pcwalton: then it should be fine
[6:02pm] pcwalton: right
[6:02pm] pcwalton: that's the important case for fast compiles
[6:02pm] nmatsakis: right
[6:02pm] pcwalton: since optimization turned on invokes a lot of whole program analyses
[6:02pm] pcwalton: which are going to invoke amdahl's law in a hurry
[6:05pm] graydon: interesting
[6:05pm] graydon: because ... I certainly *intended* to be able to tease rustc apart into function-at-a-time work for ... much of what it's doing.
[6:05pm] graydon: but became increasingly dismayed at that when considering trans tight coupling to LLVM
[6:05pm] graydon: and the sense that LLVM was generally serial
[6:06pm] pcwalton: I think it should work as long as LLVM allows us to build functions in parallel
[6:06pm] graydon: maybe I'm imagining that
[6:06pm] graydon: yeah
[6:06pm] graydon: that's a sort of key question
[6:06pm] pcwalton: LLVM types are interned
[6:06pm] pcwalton: so there's probably going to be locking there
[6:06pm] pcwalton: ditto with inserting functions into the Module, I assume it would have to lock the Module
[6:06pm] nmatsakis: not necessarily a problem.
[6:06pm] graydon: a crate gives you a parse plan for parallel front-end stuff. resolve and typeck need a little bit of inter-function chatter but I think it can be managed with messaging (making requests and waiting on them)
[6:06pm] nmatsakis: the intern table I mean, we could make that fast if we had to :)
[6:07pm] graydon: the rest ... actual lex/parse/expand, and typestate, and most of the safety / usage checks, trans .. I think should at least run source-file or function-at-a-time
[6:07pm] pcwalton: right
[6:07pm] nmatsakis: graydon: I have also been toying around with ideas that would make parallelizing much more straightforward, I think, as they would allow for read-only sharing of the AST and so forth.
[6:08pm] nmatsakis: something I wanted to bring up at some point =)
[6:08pm] nmatsakis: basically intra-task data parallelism.
[6:08pm] graydon: mhm
[6:08pm] graydon: I believe this is entirely within the realm of reason if you work with unique types
[6:09pm] pcwalton: it's actually constification here
[6:09pm] pcwalton: which is... a little tricky
[6:09pm] graydon: this is one of the things that made me very excited about moving in the direction of unique pointers
[6:09pm] pcwalton: but perhaps a good thing, because rust likes immutability...
[6:09pm] graydon: constification and uniqueness feel (to me) like two sides of a very similar coin
[6:09pm] nmatsakis: pcwalton: yes, constificaiton thought I'd prefer to make mut-ification
[6:09pm] nmatsakis: that is, make types read-only by default.
[6:10pm] pcwalton: yeah, that's basically how rust works now
[6:10pm] nmatsakis: so if you want a ptr to an object you plan to mutate, you make it a mut pointer.
[6:10pm] pcwalton: that's different
[6:10pm] nmatsakis: well, it's very similar to how rust works now but also different.
[6:10pm] pcwalton: deep mut instead of shallow mut
[6:10pm] nmatsakis: it'd be shallow mut
[6:10pm] nmatsakis: that is, @T would be a deep const T
[6:10pm] pcwalton: oh right
[6:10pm] pcwalton: deep const instead of shallow const.
[6:10pm] nmatsakis: @mut T would be "I can change the mutable fields of T"
[6:11pm] nmatsakis: anyway this is getting far afield
[6:11pm] nmatsakis: just wanted to bring up that I have been playing with it
[6:11pm] nmatsakis: graydon: I agree that with unique ptrs we could probably restructure the compiler to use tasks
[6:12pm] nmatsakis: but it would not be a trivial undertaking
[6:12pm] graydon: agreed. I think it could also be done in steps though, not boil-the-oceans
[6:13pm] graydon: anyways, I just want to point something out, a hypothetical pattern for intra-task parallelism
[6:14pm] graydon: which is: pass a unique type (possibly mutable, whatever) to an opaque, unsafe helper library
[6:15pm] graydon: the helper library runs tasks with reference-to-const-T inputs, in parallel
[6:15pm] nmatsakis: that is part of my proposal, yes.
[6:15pm] graydon: they can't mutate the T. the T stays alive for the duration of the parallel processing.
[6:15pm] nmatsakis: well, not exactly.
[6:15pm] graydon: then the helper hands it back to you when done
[6:15pm] nmatsakis: basically the idea was to have closures where all upvars are deep const
[6:15pm] nmatsakis: these can be executed in parallel safely
[6:15pm] nmatsakis: but using unique ptrs you can also give them mutable data that only they have access to
[6:16pm] nmatsakis: or you can do stuff like have a ~[int] where each helper gets a read-write view over the array
[6:16pm] nmatsakis: sorry, read-write view over a distinct portion of the array
[6:17pm] nmatsakis: (sorry, one additional wrinkle that is important: when the parallel tasks are executing, the "main" parent task that spawned them is frozen, so that it does not race with the parallel subtasks)
[6:17pm] nmatsakis: basically it blocks until they complete
[6:18pm] pcwalton: similar to PJs
[6:18pm] pcwalton: right?
[6:18pm] nmatsakis: yes but static
[6:18pm] nmatsakis: are there are other things we wanted to discuss in the meeting?
[6:18pm] pcwalton: ISTM that PJs is a good way of prototyping this
[6:18pm] nmatsakis: agreed
[6:19pm] graydon: PJs?
[6:19pm] • Yoric wonders what's a PJ.
[6:19pm] nmatsakis: a parallel javascript extension I"m working on based on the same ideas
[6:19pm] Yoric: Pyjama?
[6:19pm] Yoric: (almost)
[6:19pm] pcwalton: um... I wanted to give kudos to brson for making forward progress on libuv
[6:19pm] nmatsakis: yes!
[6:19pm] pcwalton: and it can now fetch the google home page (serially)
[6:20pm] Yoric: Nice :)
[6:20pm] Yoric: Is it just network or also local i/o?
[6:20pm] pcwalton: just network, libuv doesn't do local i/o unfortunately
[6:20pm] pcwalton: (yet)
[6:21pm] pcwalton: AIUI local i/o is harder because a lot of operations actually block internally especially on POSIX systems
[6:21pm] graydon: woah seriously?!
[6:21pm] graydon: um yes kudos of the highest degree
[6:21pm] pcwalton: it's very minimal right now I'm told
[6:21pm] pcwalton: just enough to make forward progress on servo
[6:21pm] pcwalton: but it will be beefed up considerably
[6:22pm] nmatsakis: also props to pcwalton and fzzzy for the Rust-SpiderMonkey work...I understand that's coming along well too.
[6:23pm] nmatsakis: one thing I wanted to bring up then: I have been toying with this iteration library but I'm a bit stymied by the bind syntax, which seems too clumsy.  I am considering making it lighter by omitting the bind keyword (and changing how it's implemented to use the same path as closures, which is more complete and---in particular---will work with interfaces).  I think it would be nice to have a nice iter library but I'm not sure whether I ought to prioritize work like this, which seems... not in the critical path.
[6:23pm] kib2 joined the chat room.
[6:24pm] nmatsakis: basically something like _.foo(_) would be equivalent to {|x, y| x.foo(y)}
[6:24pm] nmatsakis: the motivation being to allow us to write stuff like vec.iter(_).map(f, _).to_list()
[6:24pm] nmatsakis: where map, filter, etc, would all be generic functions that work over any "iterable" thing
[6:24pm] Yoric: pcwalton: I really wished I had some time to work on AIO.
[6:25pm] Yoric: In Rust, that is.
[6:26pm] fzzzy: Yoric: doesn't uv already have an equivalent?
[6:27pm] Yoric: Does it?
[6:27pm] nmatsakis: shall I declare the meeting at an end? :)
[6:27pm] fzzzy: i thought so
[6:28pm] brson: I hoped we would discuss our 0.2 goals and timeline, put issues on the milestone list
[6:28pm] nmatsakis: I do feel somewhat directionless.
[6:29pm] graydon: ok
[6:29pm] graydon: I want 0.2 to build more reliably on the same set of platforms, and have the libraries better fleshed out. I don't care about any other language work. happy to leave the language mostly as-we-shipped-it for 0.1.
[6:30pm] graydon: semicolons and new bind or export syntaxes, ok, maybe a bit of sugar-work here and there.
[6:30pm] graydon: library-and-tool work, though, is the looming issue in my mind now
graydon: rustdoc++, libuv++, reorganizing libstd::os and generic_os mess to actually be sane (and probably be in core), regularizing naming, picking central ifaces we want people to use longer-term
[6:31pm] nmatsakis: to that end, an iter library seems to fit in.
[6:31pm] graydon: simply for this reason: API breakage is more likely than language breakage. people aren't necessarily going to use every last feature of the language, but they are going to start calling APIs.
[6:31pm] graydon: yeah, iter is a great thing to have spent some time on.
[6:31pm] damag left the chat room. (Quit: Leaving)
[6:32pm] nmatsakis: I think we really ought to fix our memory management situation.  Box allocation is too slow.
[6:33pm] nmatsakis: I started hacking on also over the last few days.
[6:33pm] nmatsakis: what is time frame for 0.2?
graydon: I'd like to move on it pretty quick. couple months at most? regular releases >> long build-ups
[6:35pm] graydon: definitely not another 18 or 27 months
[6:35pm] jdm: let's ride the rapid release train!
[6:35pm] fzzzy: Yoric: "uv_fs_*" https://github.com/joyent/libuv
[6:35pm] linuxfood joined the chat room.
[6:35pm] graydon: well, I don't think we quite need 4 layers of beta and military discipline yet
[6:36pm] graydon: there are still not quite enough of us to be colliding that hard
[6:36pm] graydon: but it'll get worse, we should establish a pattern
shoenig: any recommendations for things to work on for newcomers to the project?
[6:37pm] oconnor0: is fixing the windows installer on the agenda?
[6:38pm] graydon: yes
[6:38pm] oconnor0: thanks.
[6:38pm] graydon: that's another reason I want to "move quick" on 0.2. I'd even be ok with 0.2 going out asap, and calling 0.3 the 2-month horizon release. 0.1 is always unlucky :)
[6:38pm] Yoric: fzzzy: ok, my bad
[6:39pm] Daeken: is anyone working on that (the windows installer)?
[6:39pm] graydon: Daeken: yes, I am.
[6:39pm] Daeken: ah, ok
[6:39pm] graydon: I mean to flip my desktop over to windows today so I can do that kind of stuff in tighter feedback loop
[6:40pm] graydon: currently doing it in VMs and remote, which is too slow, too easy to let slip
[6:40pm] graydon: windows devs are still a huge portion of the population
[6:40pm] nmatsakis: graydon: I am not sure about the "all library" for 0.2.  At minimum, I do think we want to make things run faster.
[6:40pm] nmatsakis: Faster messaging and allocation.
[6:40pm] fzzzy: fast message passing, libuv, those are the things that matter to me
[6:40pm] nmatsakis: (I do agree getting windows build working is good too)
[6:41pm] brson: shoenig: the 'easy' tag on our github issue tracker has some ideas
[6:41pm] froystig joined the chat room.
[6:41pm] graydon: ok.
[6:41pm] brson: shoenig: I suggest working on perhipheral crates, written in rust, like the core/std libs, or cargo, or external libraries, to get your feet wet
[6:42pm] graydon: this is why I asked earlier about dividing inter-release work schedules into phases (perf work, library work, etc.)
[6:42pm] nmatsakis: Right. 
[6:42pm] graydon: clearly nobody liked my idea, but .. I agree some balance is wise. I just don't want "balance" to turn into a short-hand for "let's do everything" and then regular-releases get delayed
[6:42pm] nmatsakis: well I think we should keep 0.2 on a tight leash.
[6:43pm] nmatsakis: let's pick a few priorities, do them, and then pick some more for 0.3 :)
[6:43pm] Daeken: graydon: if you want a hand, let me know.  very new to rust, but looking to jump in, and i work primarily from windows (although i'm currently tracking down an ICE compiling rustc on linux :P )
[6:43pm] graydon: that's the key point: you're always going to be disappointed by each release. you have to psychologically insulate yourself against trying to get everything you want into a release. that way is death.
[6:43pm] nmatsakis: so, I think monomorphization will fix a lot of crash bugs.  I personally encounter problems with enum alignment and other weird things regularly.
[6:44pm] nmatsakis: if we're going to monomorphize, I'd rather do that then fix enum alignment based on tydescs.
[6:44pm] nmatsakis: therefore I might argue for making it one of those few priorities.
[6:45pm] brson: I will put various things on the 0.2 list based on my judgement + this convo
[6:45pm] |wilsonkk| joined the chat room.
[6:45pm] graydon: ok
[6:45pm] nmatsakis: anyway, I know pcwalton and dherman are not here, so perhaps we should toss some stuff on 0.2 list and prune it down next week
[6:45pm] nmatsakis: ditto brson...
[6:45pm] brson: graydon: do you have specific things in mind about "building reliably" besides the windows issue?
[6:45pm] graydon: I agree monomorphization is "a priority". I just don't think (or know) that it can be done within a couple months.
[6:45pm] _wilsonkk_ left the chat room. (Ping timeout)
[6:45pm] brson: *issues
[6:46pm] nmatsakis: graydon: yes, it might be that it begins and runs in parallel, doesn't land for release.  particularly if the release focuses on the front end, conflicts ought to be minimal.
[6:46pm] nmatsakis: where by front end I am lumping in libraries, tools, etc
[6:46pm] graydon: brson: there are a couple weird bits. linux ABI tag. win32 installer. I'd like to get mingw-cross working. I'd like to get msvc working. other linux packages would be great (PPA / RPM). freebsd or pkgsrc.org. OSX homebrew or port.
[6:47pm] graydon: brson: "miserable infrastructure work" for the most part. I mean, I can do that stuff full time and still it never all gets done.
[6:47pm] graydon: I just want to keep some of my focus on it
[6:47pm] graydon: maybe only takes 1 person
[6:48pm] brson: ok. we can leverage the community for some of that too. lots of people have stepped up to do packaging work - we just have to bless it and encourage them
[6:49pm] brson: I know we have a PPA, homebrew and freebsd port
[6:50pm] brson: and homebrew
[6:50pm] graydon: great
[6:50pm] brson: oh, I said that
[6:50pm] graydon: let's link to them visibly! and/or adopt them if they feel excluded or burdened by it
</pre>
