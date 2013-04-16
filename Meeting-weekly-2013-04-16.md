# agenda

- reminder re: reviewing contents of bors queue (graydon)
- infrastructure update (graydon)
- warnings about dead assignments etc (nmatsakis)
- destructors and moved values (nmatsakis)
- gettext/iconv/unistring LGPL, libICU and unicode (again; graydon)
- selecting a meeting time for triage (graydon)
- converting `~` to `@` (pcwalton)

# attending

graydon, tjc, jack, brson, pcwalton, jclements, pnkfelix, nmatsakis

# reviewing contents of the BORS queue

- G: The queue has expanded and there are a lot of pending things
- G: Many of them could land, I imagine
- tjc: As of late yesterday, it looked like most of the things were still blocked on some question
- G: It would also be nice if bors could give a better indicate of state
- G: Probably I'll just add a note regarding who made the last comment and when
- G: This lets you see if the last comment was by a reviewer or not
- N: Maybe mention any @mentions too

# Infrastructure update

- G: Brian, Jack, and I had a meeting with rel eng concerning moving our stuff to rel eng
- G: So that we don't have to maintain it
- G: That will gradually go forward pending their ability to preserve bors etc
- G: So there may be some slight changes to the working procedures going forward

# Warnings about dead assignments

- N: I was putting in new borrow checker and found that it would be helpful if it had a more precise notion of flow control than the one put in so far. too many false errors.
- N: thing I wanted to ask: right now we have liveness code that computes when a variable is live. Does so with reverse-flow analysis. For various reasons, borrowck should be a forward analysis. Would be convenient if same dataflow algorithm used between these two parts.
- N: main point: some warnings won't be possible if I do this. for example: dead assignment warning isn't likely possible in forward analysis, requires backward analysis. I would prefer to lose these messages than preserve duplication of analysis passes.
- T: can we have separate analysis? or is that what you're discussing losing?
- N: that's what I'm discussing
- N: would be easier if we had a CFG, but we are doing this at the AST level and it's pretty grungy to even have written once, twice seems like too much
- G: note: in rustboot we just built a CFG
- P: not interested in a full lower-level IR, but a CFG is ok
- N: ok, I will look into maybe building a CFG, covering cases where we keep side tables now (possibly)

# destructors and moves

- N: something I wanted to discuss for a while, make sure it gets mentioned
- N: with respect to destructors and moves: if you have a dtor and something is conditionally moved, the dtor runs when the variable goes out of scope _if_ it hasn't been moved. This requires flags indicating whether something was destructed, and such.
- N: We've tossed around the notion of getting rid of these flags, but this would change "when dtors run". Wanted to ask if people think this is the best possible scheme or if something else might work.
- P: is it possible to transform so that we can statically insert "must be destructed" points?
- N: we could duplicate the CFG, but that's ... not ..
- J: I don't understand what you're proposing. If you get rid of the flag, when do you run dtor?
- N: on CFG edge where moving from init to maybe-not-init
- F: so it'd run earlier?
- N: yes, potentially.  For example it might run as you exit the else branch of an if, if the value had been moved on the then branch.  Personally I think this might be a bit hard to explain.
- P: I'd very much like to move away from adding fields to values
- F: I would have guessed that you would just force there to be a move on the else branch
- G: I would prefer that. a generic consume() function that move-disposes of a thing.
- N: That's another option, that'd make the destructor explicit
- P: I'm a bit worried about having to do it everywhere
- N: One possible extension I've been tossing around with (for future reference) is support for partial moves like this:
    self.left = consume(self.left)
- B: Does that interact with destructors?
- N: We don't allow you to move out of structs that have destructors
- G: Can we experiment with that? (forcing moves on all paths)
- N: I can do it once borrow checker work is done
- P: can we try today?
- N: it's a bit of work today, without a CFG
- P: let's talk about it after meeting

# getitext/iconv/unicode

- G: This is not a fun topic but I want to fish for advice here
- G: For English, the amount of unicode functionality we have now is fine, but for other languages it's not adequate
- G: Can't sort strings, normalize, and so forth
- G: No support for message catalogs
- G: Might have been optional once upon a time but people expect better unicode support today
- G: GNU has some nicer libraries and then there's ICU, which is huge but complete
- G: GNU stuff is LGPL unfortunately and thus not usable in our core libraries without forcing downstream users who statically link to be open source
- G: This is because end user must be able to patch the library, which is possible for dynamic linking but not static linking
- G: Two options: roll our own (a lot of work), try and pick up ICU
- G: ICU is MIT licensed
- F: Why don't other languages have this problem?
- G: Because their unicode support is not very good
- P: What does Go do?
- G: They roll their own
- N: I kind of feel like 20 MB...so what
- G: It may not be that bad if you statically link since you may not need all data tables
- P: Can we take a hatchet to ICU to make a minimal build configuration?
- G: I am playing with that
- G: Mainly I wanted to know if anyone thought this was just too big to consider?
- Jack: not to me
- B: Does this have to go in core, really?  Why not std?
- G: I don't especially care about core vs std, particularly once we have a package manage that works
- G: but it feels to me like something that we should support 
- B: Can ICU be compiled in kernel mode?  Would you ever use ICU in a kernel? I'm wondering how much this will burden core as we try to move core into other contexts?
- P: Looking at ICU's page, they try to convert between all manner of encoding tables, calendars, date formats, we probably don't need all of that
- P: Honestly I am mostly concerned about the compile time
- Jack: One note, everyone has to ship ICU on iOS now since it's considered a private lib
- P: Can we use the system ICU on mac/linux?
- G: I think Windows has their own alternative?
- G: We could maybe make an abstraction layer over top the system library
- Jack: it looks like QT5 binds to libICU on mac/linux but requires libicu to be installed on windows
- JC: LibICU core is only 5 MB
- ...
- P: WinNLS seems to be the windows option
- G: Probably this ought to be in the lib section on the wiki

# Weekly bug triage

- G: I want to establish a meeting time for the weekly bug triage
- G: Do people have preferences in terms of time? Time zone? Day?
- N: Earlier is better for other time zones
- B: What is the process to nominate a bug for a milestone?
- G: Just add the Nominate tag

# Converting ~ to @

- P: Niko had concerns about this because eventually we'd like to not have headers on ~ allocations
- P: But it occurred to me that since we have to trace ~ which contains @ anyhow
- P: Perhaps we could just adjust ~ so that it points after the headers?
- G: Let me just back up a few steps here
- G: We already need support for shared unique.
- G: There will always be ~ boxes that the GC must walk through
- G: However, the GC information does not have to be held in a header
- P: Can we move it outside the box?
- G: I think so, I am planning on doing that
- P: There is this annoying case with the borrow checker where sometimes it has to temporarily freeze @mut things
- G: You still need some header
- P: It has to be very cheap to twiddle that bit
- G: It can be kept in dense maps
- N: Is there a problem when you have a ~int that begins its life as a unique pointer and then becomes an @int?
- G: Most allocators have some point where they switch strategies 
- N: Normally that's a function of its type, we'd have to be prepared for any random pointer to have begun its life on the exchange heap
- G: Yes, right.
- N: It'd certainly be a useful capability, so as long as we can address these issues without burdening `~T`  allocations with headers or limiting their capabilities, I'm all in favor

