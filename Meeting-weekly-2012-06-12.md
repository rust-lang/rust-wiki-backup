# Attending

Ben, Tim, Patrick, Brian, Sully, Niko, Eric, Lindsey, Paul, Dave, Graydon

# Dynamically sized types

  * **Niko**: I think it's the nicest way to combine the vector types and function types, but recognize there's room for disagreement; thoughts?
  * **Patrick**: +1
  * **Patrick**: I want to write @vec instead of vec/@, same with functions
  * **Tim**: it's simplifying
  * **Graydon**: semantically, is it cool? also, I don't quite like the syntax; don't like the special case vec type
  * **Graydon**: can't instantiate them
  * **Niko**: right
  * **Paul**: I think that's ok
  * **Dave**: it's a familiar concept from C/C++
  * **Graydon**: yeah; guess I want to know there's good precedent for this
  * **Dave**: C is a serious precedent; they're weird at the margins with statically known array sizes, which this doesn't do
  * **Patrick**: plus C++ has the weird slicing problem, which this doesn't have; almost nothing in Rust is open -- no open classes, for example
  * **Niko**: we can avoid that problem if it should crop up; technically that's the copy constructor that's making that happen
  * **Niko**: for dynamically sized types by value, if there were open subtyping we'd have to be careful
  * **Patrick**: but we also don't have dynamically sized by value types; so we're safe in two ways
  * **Graydon**: just had a thought: syntactically, in C they omit the size from the array notation; we've talked about a fixed-size syntax inside the square brackets; if we have a way to write fixed size, dynamic size is same syntax but with underscore
  * **Niko**: tried to make that work many times but failed to make it work; I'm not opposed
  * **Graydon**: does anyone object to the concept of dynamically sized types?

(crickets)

  * **Graydon**: ok, so it's mostly just a syntax question; we'll fuss around with that later

# Patterns

  * **Niko**: binding-by-reference in patterns; I emailed the list: right now we bind by reference always but the types hide that from you -- probably the worst spot in the design space to be; it's confusing and non-obvious, but efficient
  * **Niko**: originally wanted types to reflect that, but sometimes you want not a pointer but move or copy; so idea was to let you specify explicitly
  * **Paul**: is this like binding modes, analogous to argument modes?
  * **Niko**: kind of, but won't be 10 million of them
  * **Niko**: basically two: copy or make reference
  * **Dave**: three right? move too?
  * **Niko**: move falls out of copy
  * **Niko**: think of it as by value or by reference
  * **Patrick**: like + vs &&
  * **Eric**: what about instead of modes, just keep it by reference by default but you can copy explicitly
  * **Niko**: can't move then b/c reference still exists
  * **Graydon**: totally fine except syntax
  * **Graydon**: if all pattern variables are references, put unary copy in to get copy
  * **Niko**: I wanted to unify patterns in let and alt; this would not achieve that
  * **Niko**: probably don't want to write let copy x = y
  * **Niko**: we could also not unify them
  * **Patrick**: I like the conceptual simplicity
  * **Graydon**: I just don't like the unary *
  * **Niko**: there's no pattern for unsafe pointers, and secondly I think we could move unsafe to a special region; * would just not be a pointer syntax
  * **Dave**: why doesn't & work?
  * **Niko**: I think * is kinda ugly but I'd get used to it; it's consistent in the sense that a pattern is what you have to type to get the value back
  * **Dave**: oh yeah, it's destructuring which uses *structuring* syntax
  * **Niko**: reason & doesn't work is we'd want patterns that work on regions, and & would be for that
  * **Graydon**: I still kinda want default in destructuring to be by reference, because it's more efficient
  * **Niko**: I hear you
  * **Graydon**: we'll poke at that a little more; but I'm glad you're working on this
  * **Paul**: Question: was there some plan for removing argument modes? would that apply to this also?
  * **Niko**: no. plan for removing argument modes was to go to by-value; you own value but can pass region pointer (owning a region pointer isn't significant) but that doesn't really solve this question

# Maxmin classes

  * **Patrick**: proposal on my blog; paring down classes to be more minimal; basically nominal records with an impl
  * **Patrick**: that wasn't done at first b/c:
   * wasn't clear whether you could mix traits into impls (you can now)
   * privacy was the big thing that wasn't handled (we can do same as Go, have private fields be scoped to module)
   * destructors are a little tricky (for now let's assume they're a separate form you would write as part of the class; I'd eventually like to make them ifaces but depends on coherence)
   * constructors (can just be record-literal syntax)
  * **Patrick**: upshot: not much left to classes except nominal records
  * **Patrick**: I think this is nice because it reduces number of moving parts in the language; wanted to get people's reactions
  * **Eric**: the one thing I really like is you can move out of the self-pointer
  * **Patrick**: yes, flexibility of the self-pointer; not tied to the class but the individual method
  * **Niko**: (explained what the central difference is, Dave still didn't get it)
  * **Brian**: maxmin classes is about turning classes into something closer to our existing impls, right?
  * **Patrick**: actually about turning classes into nominal records and then using impls to add methods to them

# Eliminating modes

  * **Graydon**: broader strategic question: is there anyone who doesn't think we're gonna eliminate modes and use region pointers for that task?

(crickets)

  * **Graydon**: ok, scheduled for demolition and you can just assume that
  * **Dave**: Graydon, what's your feeling about maxmin classes?
  * **Graydon**: I'm fine; seems simpler, I'm not that big on OO so I don't much care
  * **Brian**: when are we getting rid of modes?
  * **Niko**: soon; I have a plan for how to go about it
  * **Graydon**: just a plan of attack problem; is there substantial remaining proof that you can do it?
  * **Niko**: no, I'm positive; the fact that borrowck is in convinces me we can do it
  * **Dave**: (rejoices internally)
  * **Patrick**: what do we do about ++ mode?
  * **Graydon**: I don't even remember what it means
  * **Patrick**: it's unsound; doesn't run destructors or copy constructors properly so it can crash
  * **Niko**: lemme spell out what it does: copies all bytes of value onto stack but doesn't transfer ownership; @T copies pointer; record copied entirely on stack, but all data not owned; unsoundness if record has mutable fields and you mutate, weird stuff happens; you're just mutating your local copy but you don't own it, if they're unique etc etc
  * **Graydon**: exists just for FFI?
  * **Niko**: used by default for all word-sized types; reason is efficiency optimization to avoid refcount traffic & cloning uniques
  * **Niko**: borrowing feature of regions was designed to address this problem, but have to decide if that's what we wanna do
  * **Niko**: by-copy mode creates local copy and passes pointer
  * **Graydon**: someone said "we have to be able to do ++" but why?
  * **Patrick**: I think it's the FFI, right?
  * **Niko**: all FFI wants is by-value
  * **Patrick**: so it wants ++
  * **Niko**: no it wants C value, ++ is Rust value
  * **Niko**: ownership is ill-defined in C
  * **Patrick**: just wanna make sure we don't break FFI
  * **Niko**: I want to merge + and ++: copy data on stack; if you pass an @ or ~ it belongs to callee (if caller owns, use a region pointer)
  * **Patrick**: it's a move?
  * **Niko**: not a move, it's a copy
  * **Patrick**: would increment refcount?
  * **Niko**: y
  * **Niko**: would also be a deep copy of unique, but no-implicit-copies prevents that happening automatically
  * **Patrick**: I'm fine with that
  * **Graydon**: me too; if there's a wrinkle in something C wants purely for the sake of C, then the fact that it's extern ABI is what you hang that off of; change what you're doing by argument passing
  * **Graydon**: ok, so is there any concern about the loss of ++?
  * **Patrick**: I think I'm convinced this works
  * **Niko**: my plan of attack: impl an analysis that if we went to by-value, where would we break or make expensive copies, and try to convert those to explicit regions and then make the switch
  * **Niko**: basically add a lint pass
  * **Graydon**: user-facing reason for lint pass as well
  * **Graydon**: is that on your near-term?
  * **Niko**: I'm working on a paper on the region stuff, but I'm still writing code so that's basically my next step

# Release 0.3

  * **Graydon**: there's a bunch of open stuff right now; we could do a triage meeting now or later
  * **Patrick**: Brian's feeling was we should punt on everything except resolve
  * **Patrick**: hard to justify shipping with current resolve b/c it bites ppl way too often
  * **Graydon**: plausible
  * **Patrick**: I've been working on rewriting it; I have almost all values in compiler resolving
  * **Patrick**: types are getting there; hopefully this week?
  * **Graydon**: curious about one thing: is this derived from marijn's stuff, or started again?
  * **Patrick**: started again; need to have another conversation about import
  * **Patrick**: turns out I don't really know a way to resolve the current system; it turns out to be incredibly complex to get right; old resolve didn't handle it properly; mine's conservative under-approximation, should avoid bugs but can get stuck in presence of certain kinds of cycles
  * **Patrick**: at some point we should think about simplifying; don't even know that it's coherent
  * **Graydon**: two small points: previous discussion we've had involve changing syntax slightly; looking at doing that?
  * **Patrick**: first I want to get it working w/o changing anything, then I'll change the way export works to use pub etc.; at the same time we can update classes to use the same rules
  * **Graydon**: Brian believes we need to focus on resolve for 0.3? 0.3 should be resolve + whatever's already in?
  * **Brian**: yeah, that's fair; we're overdue by 2 weeks
  * **Graydon**: yeah, it's a random date but might as well
  * **Graydon**: slight concern: not clear that rewritten resolve is what we're going to settle on; concerned we're telling users "here's a new system" and gonna change again
  * **Dave**: well, not really user-facing change, just bugfixes
  * **Graydon**: gutcheck: do you really think it's less buggy?
  * **Patrick**: yes. fundamental difference is new resolve is strictly conservative; never makes any decisions that it doesn't fully know implications of; old resolve was order-sensitive
  * **Patrick**: you can get errors, but it tells you exactly the lines where the errors occur
  * **Patrick**: if we do change the syntax, it can build on the existing algorithm; also 4x faster
  * **Graydon**: cool, glad you did that
  * **Graydon**: seemed like there were source-level bugs as well as linkage-level bugs
  * **Patrick**: may be metadata bugs
  * **Graydon**: export tables not being built properly or something like that?
  * **Patrick**: haven't touched that, but after resolve metadata is probably hairiest part of compiler; due for rewrite
  * **Patrick**: I know you've had ideas about changing encoding format; I'd be ok with that
  * **Patrick**: I'd be fine with auto-serializing Rust data structures for the metadata
  * **Graydon**: I'd be fine with that; EBML is a little less future-proof, but I totally want to use auto-serialize
  * **Patrick**: then our schema would be documented; our schema is crazy
  * **Graydon**: it's free-form, yeah
  * **Niko**: should be easy to virtualize
  * **Graydon**: yeah, I know what we need to do; looks to me like most promising is Apache Avro; kinda weird but I kinda like it
  * **Graydon**: alternatives:
   * MessagePack is isomorphic to JSON but binary; lots of tag redundancy but super simple
   * protobufs kinda in the middle; requires you to keep numerical identities from version to version; not super easy to maintain
   * Avro: every serialization blob has prefix on, written in JSON, that describes schema; thing that comes next is packed binary; no tags, super dense content
  * **Graydon**: one way or another, EBML doesn't feel like the right approach for us
  * **Niko**: about resolve: we don't know what's causing the bugs; could be anywhere; we should probably at least make sure they have a common tag and try to test them
  * **Graydon**: what else is near ready?
  * **Lindsey**: good chance inferred integer literal suffixes will be
  * **Patrick**: that's actually huge
  * **Dave**: users will be very happy
  * **Graydon**: region pointers were on the second release; eliminating modes will be a big long slog?
  * **Niko**: don't really know; might not be but don't want to commit
  * **Patrick**: another journey: eliminate implicit copies; if you uncomment that warning, there are hundreds of places where we're copying e.g. strings; can't be good for perf
  * **Graydon**: a constellation of work here: includes:
   * reflection
   * moving to ifaces
   * str and vec work
   * dyn sized types
   * eliminate modes
   * make everything use references properly
  * **Graydon**: every time I touch one, they all seem to depend on each other; doesn't feel like that constellation is going to be done for 0.3; that's the next time 'round

# Roadmap

  * **Graydon**: let's set up a separate meeting this week
  * **Dave**: I'll schedule it

# Repo policy

  * **Tim**: were we going to talk about pushing policy?
  * **Lindsey**: Patrick, can you explain?
  * **Patrick**: I propose `incoming` -- works well in Firefox, a branch you can push to, once a day, last green commit gets merged to master; don't have to watch bots after you push; if you push and it breaks, somebody will back it out
  * **Patrick**: nice in Firefox b/c tests take a long time, and Rust tests take a long time
  * **Tim**: one concern: I push and tests break, you back it out, and I don't notice; have to keep polling
  * **Patrick**: true. helps if when you push, you put an issue #, whoever backs out, put a note in the issue
  * **Paul**: that would create a lot of dummy issues
  * **Patrick**: that's true
  * **Dave**: just if you back it out, tell the committer (instead of registering in issue)
  * **Lindsey**: will we have rotating sheriff duty?
  * **Patrick**: I've been doing it; it's not a full-time job like it is for Firefox
  * **Niko**: Graydon once proposed that you ask rustbot to pull from your branch, but that requires engineering
  * **Patrick**: probably ideal
  * **Graydon**: I'm supposedly doing all the automation and still working on compiler features; that's where I'd like our developer flow to be; humans shouldn't be part of the process of watching machines; machines can do that fine!
  * **Graydon**: this feels like policy as a band-aid, but if we need band-aids that's cool
  * **Sully**: we'd be merging incoming to master? that'll be lots more merges
  * **Niko**: why would we merge? should always be fast-forward, right?
  * **Eric**: yeah
  * **Patrick**: in FF they merge both incoming and master both ways
  * **Brian**: that's terrible!
  * **Graydon**: second merge is a no-op; just resetting branch pointers to same point
  * **Graydon**: my feeling is, I don't want to get in habit of assuming tests are too long to run. a slippery slope of evil to turn stuff off to make it go faster
  * **Tim**: also if you run tests locally could be bugs on other platforms
  * **Patrick**: inevitable we'll have tests that take forever to run. that's good! it means we have good tests! you won't realistically be able to watch all the tests to get work done, but we'll have to deal with this problem
  * **Eric**: at MS we did buddy builds -- send patch to someone else to have them build it for us
  * **Patrick**: I feel like bots should handle that
  * **Graydon**: all of this should be automation. scripts are currently wedging themselves; needs engineering resources we don't have; band-aid wise, this is ok
  * **Dave**: so policy is:
   * push to incoming
   * sheriff backs it out and alerts the committer
   * once a day, merge incoming to master
  * **Brian**: how about committer does the backout?
  * **Patrick**: no, sheriff will be accountable for it
  * **Sully**: I don't completely understand what we're gaining; if everyone is working on incoming, what's the difference?
  * **Brian**: yeah, just renaming master to incoming
  * **Lindsey**: why not just do this for master, no incoming branch?
  * **Tim**: maybe Patrick could write up in the wiki what procedure is, and we can discuss
  * **Graydon**: need a policy for people working off of master but pushing to incoming -- rebase?
  * **Paul**: let's call the stable branch "green"
  * **Graydon**: we update to nearest green from master?
  * **Niko**: it's just a lagging master
  * **Patrick**: let's back up. we want to avoid having to babysit landing on master. question: how much do we care about master burning, for short periods of time?
  * **Graydon**: second question: policy to backout anything on fire?
  * **Eric**: say it's up to committer?
  * **Patrick**: no, that will cause delays
  * **Niko**: just depends on whether they're available; if they're not around, back it out for them
  * **Patrick**: yeah, if it's burning, it's your right to back it out
  * **Graydon**: yeah. incoming happened on firefox b/c dev env consists of so many people that finding people is too hard; we're not at that point
  * **Graydon**: strict policy is you unblock within 20 minutes or we back you out
  * **Brian**: I totally regret this master/green thing. I think it's wrong. master is branch of record. that should be the one that's guaranteed to be stable and building. let's do what patrick said originally, call them master and incoming
  * **Graydon**: one thing I'm nervous about: we really don't like criss-cross merges that go back in time. can we make sure incoming is just an extension of master?
  * **Patrick**: that implies don't push to master
  * **Sully**: outcome for me: don't ever use master, just use incoming
  * **Patrick**: yes, only sheriff pushes to master and only when they're green
  * **Paul**: if it's on fire you can pull from master
  * **Dave**: what about pull requests? newcomers will go to master
  * **Graydon**: github has a concept of integration branch. can we use that?
  * **Graydon**: anyway, please convey to all community members with push access, no more pushes to master unless you're sheriff
  * **Lindsey**: put this on wiki page and send to rust-dev
  * **Paul**: we probably have a now-out-of-date wiki page on this
  * **Lindsey**: from this point, nobody but sheriff pushes to master
  * **Graydon**: sheriff will eventually be a robot
  * **Brian**: who's the sheriff?
  * **Patrick**: I have been and can continue to be
  * **Graydon**: we'll make a schedule
  * **Lindsey**: question: do we have a design finished for traits? supposedly what I'm working on
  * **Patrick**: no, we should discuss -- full-timers + you
  * **Dave**: offline conversation, but you should definitely be part of it
