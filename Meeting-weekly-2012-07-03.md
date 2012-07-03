## Attending

Ben, Brian, Eric, Graydon, Lindsey, Patrick, Paul, Sully, Tim

## Bug tracking

  * **Graydon**: github's issue tracker lacks bug dependency tracking
  * **Graydon**: don't want to switch to bugzilla yet, but perhaps eventually
  * **Eric**: github's is more lightweight
  * **Graydon**: is the bug janitor schedule working?
  * Interns: *look at fulltimers*
  * **Graydon**: perhaps the interns should take on bug janitorial work

## 0.3 status

  * **Patrick**: I pushed resolve3, not on by default, still in progress, feel free to chip in
  * **Patrick**: use -Z fast-resolve
  * **Patrick**: caveat: on very small programs, can be slower 10x; might slow down tests; likely to improve
  * **Graydon**: I'm comfortable shipping it
  * **Patrick**: there's a disturbing amount of code that relies on original resolve bugs
  * **Patrick**: the old one probably will be removed today

## Macro sigil change

  * **Patrick**: removing the sigil: no
  * **Patrick**: Hash is heavyweight, prefix is weird especially if your macro is module-qualified (`io::#println` vs. `#io::println`)
  * **Patrick**: Postfix-! has problems with assert
  * **Patrick**: but it looks lightweight; since there's one on the frontpage it should look lightweight
  * **Graydon**: who likes `#`? (**Ben**, **Lindsey**)
  * **Graydon**: who likes `!`?
  * **Patrick**: I definitely think macros should be the last resort, what you reach for when nothing else works
  * **Patrick**: I'm mostly concerned with the front page, `#fmt` looks like a dead spider (https://gist.github.com/3040867)
  * **Ben**: postfix-$?
  * **Graydon**: I wanted to hold on to $ for interpolation, and isn't it reserved already?
  * **Graydon**: lexeme balancing - simple version of macros that gives balanced token trees, that's it
  * **Paul**: two possibilities - perl style (\ special character, escape delimiter), python style (no special character, no way to escape)
  * **Lindsey**: I like writing #..[..] for debug messages because it doesn't look like the rest of the code, but for printf it's different
  * **Graydon**: what about when people want to use macros for new control structures? as compared to the lambda syntax we already have
  * **Brian**: we do have macros to do control structures, e.g. select (it's basically like an alt)
  * **Patrick**: "unapply" allows you to overload pattern matching, so you can do regexes, etc.
  * **Graydon**: "Do they have an operator evaluate that you can overload??"
  * **Patrick**: I'm happy sticking with # for macro syntax
  * **Paul**: it's more important for select/alt/etc to look pretty than for less important macros to look ugly
  * **Ben**: "I DISAGREE!"
  * **Eric**: having weird macro syntax makes syntax highlighting hard; with macros you can build your own binding forms; in order for an ide to do it right it has to understand the macro language
  * **Patrick**: the problem with assert is people write assert!(x) and it looks like assert (!(x)), but this is not a problem unless assert becomes a macro
  * **Graydon**: that will probably happen; fail is probably going to become a library call too
  * **Graydon**: so how do we decide which sigil to use?
  * **Sully**: I have dice...
  * *general laughter*
  * **Graydon**: so writing balanced tokens between brackets... is "select! {...}" going to be legal?
  * **Paul**: long-term goal: we'll have a set of expressions surrounded by parens beforehand
  * **Graydon**: ok, so a macro invocation is a macro name followed by a balanced token tree
  * **Graydon**: We should stick with # and revisit it if it looks like there are too many dead spiders on the page
  * **Eric**: is it awful to allow both for a while?
  * **Paul**: use # for old macros and use ! for new macros, help for migration
  * **Ben**: if we use ! for macros, will we change bang for negation?
  * **Patrick**: no, no need (if it's a unary operator); it's unambiguous
  * **Graydon**: we're killing too much time on this; let's introduce a new one with ! and see how it goes
  * **Paul**: they will live together in harmony, until we exterminate one of them

## Buildbot

  * **Graydon**: I spent the last few days fighting; convinced that it will work as a replacement
  * **Graydon**: it has five abstractions that all line up together
  * **Graydon**: machines can be thrown into a pool
  * **Graydon**: we'll do 0.3 and then move to the new one
  * **Sully**: does this mean that rustbot stuff is all going away?
  * **Graydon**: yes. rustbot is old & not as good; there will be a new irc bot
  * **Sully**: do we have something that will make perf graphs?
  * **Graydon**: not automatically; we'll have to put together a substitute

## Channel contracts/pipes

  * **Eric**: pipes are ridiculously fast, 16-17x faster than ports/chans that are in there now
  * **Graydon**: great news!
  * **Eric**: graph500 20% speed increase
  * **Sully**: what makes them faster?
  * **Eric**: no locks on the fastpath
  * **Lindsey**: and this is possible because the protocol is typechecked?
  * **Eric**: yeah, also 1-sender 1-receiver
  * **Eric**: what do people think of me landing & transitioning?
  * **Sully**: syntax?
  * **Eric**: currently you write in a language that the protocol compiler compiles to rust code that you then use
  * **Graydon**: I'd like syntax extensions to be the way we move forward on it; don't want to land something that works one way and have people start using that only to change it later
  * **Eric** rebases pretty much daily & is a git ninja
  * **Eric**: running into expressivity issues in the language; usability is painful, hard to get a resource into a closure and be able to move it out
  * **Eric**: there's some gymnastics each time you spawn a task: `~mut option<...>`
  * **Eric**: copying isn't a big deal because of two endpoints, but if you want to do many-to-one...
  * **Eric**: we might end up needing copy constructors in the language to do this nicely
  * [**Graydon**'s face looms on the camera...]
  * **Graydon**: can you get the current awkwardness into syntaxext form and then land it? are expressivity problems preventing it from being usable?
  * **Eric**: getting a pipe across a task boundary requires 6 loc
  * **Eric**: going to need some help from the language to make it usable
  * **Graydon**: we'll do it; this is exciting

## libcore/libstd blurring, doubly-linked lists, moving task.rs to libstd

  * **Ben**: we want to implement linked failure, other things in libcore
  * **Ben**: already added doubly-linked list in libcore to make this possible
  * **Brian**: I would prefer to have it in libstd
  * **Ben**: actually, I suggest moving task.rs to libstd
  * everyone: hmm...
  * **Graydon**: the original justification for the libstd/libcore split was that libstd would be the library that it's ok to have be big, and libcore has to stay small for e.g. embedded systems (and so that you don't have to put `use std;` at the beginning of a trivial program)
  * **Graydon**: my feeling is that data structure code isn't huge, so more common data structures should be in libcore, particularly if frequently used
  * **Sully**: is there an issue wrt snapshotting?
  * **Graydon**: yes, snapshotting does figure in to this
  * **Graydon**: original justification for the libstd/libcore split was https://mail.mozilla.org/pipermail/rust-dev/2011-December/001037.html
  * **Graydon**: it's still early days, could be the case that a fair amount of libstd gets used in every nontrivial program
  * **Graydon**: all this aside, the runtime library is becoming its own crate and runtime and core are going to wind up being their own library
  * **Graydon**: **Ben**, are you ok with just moving more basic data structures into core?
  * **Ben**: ok, just concerned that it seems arbitrary which data structures get to be in core and which don't
  * **Graydon**: queues, maps, lists, vectors should be in core
  * **Ben**: ok
  * **Ben**: what about @ vs. ~ versions of doubly-linked lists?  I need both
  * **Graydon**: this is a rust-specific concern
  * **Brian**: how is a unique doubly-linked list even possible?
  * **Ben**: it's unsafe
  * **Brian**: we shouldn't expose that
  * **Graydon**: good point
  * **Ben**: my hope is that the interface will be safe and it won't hand out unique pointers to its innards
  * **Ben**: can put data in, get data out
  * **Tim**: are you basically encapsulating the unsafety?
  * **Ben**: yes, also the entire list is noncopyable
  * **Graydon**: why does it need to be a list?
  * **Ben**: could also use a dvec, could have made operations O(1) with dvecs (as they are with dlists), but it would have been O(n) space in # of tasks
  * **Graydon**: the O(n) space can be fixed but it's tricky
  * **Ben**: let's discuss later
  * **Graydon**: ok
  * **Graydon**: anyway, it's ok to move more container data structures into core
