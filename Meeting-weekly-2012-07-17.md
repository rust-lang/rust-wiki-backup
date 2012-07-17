## Attending

Niko, Eric, Elliott, Paul, Dave, Graydon, Tim, Ben, Patrick

## ICFP Programming Contest Report

  * **Paul**: went pretty well using Rust for our entry
  * **Paul**: one other Rust entry, Jed Davis, who picked it up very quickly
  * **Paul**: we definitely got hit by compiler bugs, but we were able to make use of both speed and expressivity (particularly higher-order functions)
  * **Eric**: all the best parts of functional languages, but you could also be aware of cost and try to get better perf
  * **Dave**: what didn't go well?
  * **Eric**: biggest complaint we heard was complexity of the memory model
  * **Niko**: whose complaint?
  * **Eric**: that one came from Josh Wise (joshua_)
  * **Niko**: I think a lot of problems will go away when we convert alt to copy by default
  * **Niko**: and using @ all over will put you back into a typical functional style
  * **Eric**: the extra time we were allotted didn't help; got best results in first 10 seconds; so we did a bunch of different searches in parallel
  * **Paul**: biggest challenge is no one has willpower to go to sleep, so a lot of code written by groggy people

## Purity Inference

  * **Tim**: I was wondering whether we still needed this inference. would borrow-checking get use from it? unlike typestate doesn't seem that useful; seems like you'd want to declare your intent
  * **Niko**: I default to less inference until we're forced into it; borrow check is probably evolving; some day I'd love to eliminate use of purity altogether; so I'd rather not have too much infrastructure depending on it
  * **Niko**: we need a safety check, so borrow check is what we have now, but I'm not attached to it specifically
  * **Graydon**: I sort of thought its presence was essential; in what sense does purity play a role in it?
  * **Niko**: I've been working on a document explaining it; gotten lost making it perfect
  * **Niko**: way borrow check works right now: 3 critera:
   * stuff on stack
   * stuff that's not
   * fallback
  * **Niko**: if stuff not on stack but immutably rooted in something whose lifetime we know, it's ok; if there's something that the only thing you do with it is pure functions, then it's ok
  * **Niko**: only place where we really depend on this is vec::len; we do this all over
  * **Niko**: this could just go away if we switch over to boxed vectors or dvec, but at the moment it occurs semi-regularly; there's like 100 cases
  * **Niko**: not something we need to solve today, that's just where it's used
  * **Graydon**: no strong opinion, just "less machinery generally better"
  * **Graydon**: still don't quite understand why that case depends on purity
  * **Graydon**: if borrowed pointer not mutable (const), why would caller assume anything can happen in callee?
  * **Niko**: if data lives in shared box, the callee might have access to the shared box
  * **Graydon**: but shared box is immutable
  * **Niko**: in case where shared box is mutable, it matters
  * **Graydon**: I didn't realize that happens a lot
  * **Niko**: about 100 times
  * **Graydon**: when you borrow, do you increment refcount?
  * **Niko**: yes. if shared vector, no problem; just root the vector for lifetime of borrow
  * **Graydon**: this would happen if you had a uniquely-owned vector in a mutable field of a record you pass by reference
  * **Niko**: yes.
  * **Graydon**: ok, I can see that being a problem
  * **Graydon**: anyway, if inference isn't buying us anything, let's remove it
  * **Niko**: we don't even have it, just a bug for implementing it; I think we should close the bug
  * **Graydon**: I think that's ok
  * **Tim**: right. made more sense when we had typestate; now it just doesn't

## Any Type

  * **Tim**: my question was whether that's something we're still interested in; it's been open for two years
  * **Graydon**: yeah, language has changed a lot since then; my gut sense is this can be done now in libraries; it would be totally unsafe if we did it in the language
  * **Niko**: just downcast
  * **Eric**: can you cast to copy trait?
  * **Niko**: no
  * **Paul**: we could have a kind-level alt
  * **Niko**: only thing you could do would be downcast
  * **Graydon**: having this in bug tracker is too heavyweight. too many things that are "wishlist or less" in bug tracker
  * **Tim**: there's an example in comments where it might be useful, but people haven't been screaming for this feature like others
  * **Graydon**: b/c we have abstract interfaces and can box anything, you can kind of just write a trait that serves the role that the any type would serve
  * **Graydon**: I don't feel like it's a really compelling need anymore; it was stronger when we didn't have bounded polymorphism
  * (general agreement)
  * **Graydon**: in my mind this comes up more in the reflection interface
  * **Graydon**: in my mind it'd be a void * + a type descriptor packaged up in a record, but the reflection interface is still pretty young; I'd just mark this as a duplicate of the reflection interface
  * **Tim**: sure
  * **Niko**: good point
  * **Tim**: where's that filed?
  * **Graydon**: "replace shape code with visitors" is the phrase to look for

## Incremental Recompilation

  * **Tim**: I was gonna start working on this; I hope it means caching object files and not compiling things that didn't change
  * **Tim**: first part is figuring out what "dependency" means
  * **Tim**: if anyone changes something that changes linkage model or adds/changes assumptions, please talk to me about it
  * **Graydon**: language has a lot of source-level constructs that can depend on each other; we have to express those dependencies at linkage-level constructs
  * **Graydon**: not clear the things coming out of the compiler are... well, certainly they're not bug-free; more generally I don't think we have a clear map from source constructs to link-level constructs
  * **Graydon**: not as clear what inter-crate relationships mean
  * (interrupted for cat management)
  * (Tim is replaced by a cat)
  * **Graydon**: Tim, your cat solved all our linkage problems
  * **Tim**: awesome!
  * **Graydon**: level of formalism in compiler is much higher than level of formalism at the linkage level
  * **Graydon**: I think a lot of prerequisite work for doing anything here will require fixing the compilation model
  * **Graydon**: better working with cargo, better versioning
  * **Graydon**: need to formalize what we think should work
  * **Graydon**: Tim's interested in classifying, writing up relationships between compilation units and source files
  * **Graydon**: want everyone aware that's what Tim is doing; I'll be working on the driver, modernizing it, cleaning it up
  * **Graydon**: anyone with questions/comments/beliefs to talk to Tim or me about those sorts of things
  * **Graydon**: we've all messed with this stuff; I've regularly been in the middle of trans and mucked with the linkage model as a result of some need I had
  * **Niko**: sounds great, makes sense; just one thing: should we try to brainstorm/discuss what are the dependencies? I can think of a few that may not be obvious
  * **Niko**: file or item granularity? item seems nice but maybe too much
  * **Niko**: there are dependencies in the type system, some of which are obvious, some of which less so
  * **Niko**: bug? wiki page?
  * **Tim**: wiki page sounds good. figured I'd write something down that's probably wrong and then we can start discussing and people can correct my thinking
  * **Graydon**: Tim's a little new to this broad structure of the compiler, and wanted a couple days to study it and make notes; we'll then have a few conversations where people reconcile their mental models
  * **Graydon**: probably a lot of assumptions baked into each module that people don't know about
  * **Patrick**: also, reachability is something that's tied in with these concerns
  * **Dave**: what's reachability?
  * **Niko**: it's a pass in the compiler
  * **Patrick**: we're always fixing bugs in it
  * **Niko**: it's basically a problem of DRY -- reachability has its own definition which is not validated or checked against what the rest of the code does; if we could try to factor things so that when you add a link between modules, you cause a bug that would force reachability to be fixed
  * **Patrick**: wonder if we can fuzz reachability
  * **Niko**: maybe that's easier
  * **Graydon**: Niko's got a good point: need to avoid distributing knowledge between disparate parts of compiler; we have several parallel vocabularies talking about similar things

## Bug Policy

  * **Niko**: personally, I feel overwhelmed by the issue tracker at the moment, more than ever before
  * **Niko**: came up earlier in meeting offhand
  * **Niko**: what's hard for me is unclosable bugs: "I'd like it better if this method called that" or whatever
  * **Niko**: encourage people to do RFC's on mailing list rather than file a bug
  * **Niko**: that's one corner of how to get things under control
  * **Niko**: wondering if people have thoughts on this
  * **Tim**: lack of priorities on GitHub is probably part of the problem
  * **Niko**: true I guess; having fewer bugs is still helpful, although deprioritizing would help
  * **Patrick**: we currently have `vague` but problem is that's a negative term so people avoid it
  * **Graydon**: supposed to be used to say "I would like to close this bug"
  * **Patrick**: so also implies we should close it, making it even more negative
  * **Patrick**: maybe something more neutral like "investigation" or something
  * **Niko**: couldn't we just *not* have that as a bug?
  * **Dave**: you want to keep a todo list *somewhere* of "things to investigate"
  * **Paul**: wiki?
  * **Dave**: well, bug tracker is easier to keep in shape than wiki
  * **Patrick**: I've gotten mileage from investigation bugs
  * **Eric**: just close them?
  * **Patrick**: closing is a blunt hammer; once they're closed no one will ever look at them again
  * **Graydon**: we're just not doing that
  * **Graydon**: this is important and comes up in almost every meeting; couple reminders: we do have prioritization labels
  * **Graydon**: I'm really only concerned about shipping `wrong` and `crash` bugs, and completions
  * **Graydon**: we only have a couple hundred *bad* bugs
  * **Graydon**: maybe we need clearer categories; I'm willing to recategorize
  * **Graydon**: when I look at labels I see a certain amount of clumping; 162 enhancement requests. we can pursue making the compiler more correct or adding new features
  * **Graydon**: I agree that `vague` is not the kindest label
  * **Tim**: is meaning of tags documented somewhere?
  * **Graydon**: it's on the wiki; I wrote them all down
  * **Niko**: I didn't understand that page
  * **Patrick**: could we rename wish-list to investigation? wish-list is another negative; maybe we should more aggressively tag as wish-list, but "investigation" is less loaded
  * **Graydon**: sure, definitely don't want anything derogatory
  * **Graydon**: don't want judgment, just a way to make it easier for people to prioritize their work
  * **Graydon**: here it is: on the wiki under issue tracking development policy
  * **Tim**: I like the idea of encouraging people to submit everything and us be responsible for prioritizing it
  * **Graydon**: that's the bugzilla tradition; Niko have you done other Mozilla projects?
  * **Niko**: no
  * **Graydon**: Mozilla has a weird tradition with bug trackers; we tend to file a bug for everything; Bugzilla has a really good metadata system; other than tagging handles just about everything better than GitHub issues
  * **Graydon**: gonna be hard to deinstitutionalize the concept of using the bug tracker for development thinking (maybe I've also become accustomed to it myself); I don't mind allowing numbers to get large as long as filtration works
  * **Graydon**: key is making sure the filtration works
  * **Graydon**: raw count is less of a concern to me
  * **Niko**: kind of feel like we're not able to use the tags well; I didn't understand them as well as I now do, so maybe that's part of the problem
  * **Paul**: one problem is many bugs are untagged
  * **Niko**: inability to search for things that are *not* tagged is a huge problem
  * **Graydon**: I've been screaming at GitHub; such a *tiny* feature to add: just an *intersection* rather than *union* of tags
  * **Paul**: so what is integration with GitHub tracking buying us? you have closes references
  * **Patrick**: click on issues in commits, integrated with pull request system, it's what people on GitHub expect to use, integrated with GH login system so you don't have to login to bugzilla
  * **Patrick**: we could build a tool that lets you intersect labels
  * **Graydon**: there's another interface that lets you do that
  * **Patrick**: the Heroku one?
  * **Graydon**: y, but it's a whole 'nother interface
  * **Patrick**: maybe someone could write a Firefox addon for it or something
  * **Graydon**: there is an API so to some extent we could take matters into our own hands
  * **Graydon**: looks like there are several different alternate interfaces for GH issues on Heroku
  * **Niko**: I'll try to use labels more and withdraw my objection, but at the moment it's just this big mess of stuff that doesn't help me track what I need to do
  * **Graydon**: can you use assignments? assign things to yourself?
  * **Niko**: too many things currently assigned to me; I'll clear it out, and that is helpful
  * **Graydon**: you can intersect using assignment and milestone
  * **Niko**: what to people think about making labels for us, signifying "I'm interested in this bug not working on it right now"
  * **Eric**: add yourself to comments?
  * **Niko**: oh, ok
  * **Patrick**: you can create a text file where you copy/paste bug URL's
  * **Niko**: blech
  * **Tim**: yeah, just comment on the bug; we don't have too many comments on bugs
  * **Graydon**: don't want to shoot this down, Niko. were there specific things you wanted to try?
  * **Niko**: I did want to encourage people to keep bugs to something that's actionable, or at least more specified; but seems people like those kind of bugs
  * **Graydon**: I hate those. gimme a test case!
  * **Niko**: if we label those `investigate` maybe it's better
  * **Patrick**: a lot of that came about when eliminating FIXME's
  * **Tim**: in future, think more about what you create as a FIXME
  * **Niko**: there's this two-phase commit problem: I'm working on code, maybe it'll be fixed by the time I commit...
  * **Graydon**: just put the FIXME in, when you do make check it'll halt
  * **Niko**: breaks compile
  * **Graydon**: it'll remind you!
  * **Niko**: but I'm not done when I'm doing make check
  * **Niko**: I'd prefer a push hook or something
  * **Dave**: you can do push hooks with GH, I've written one
  * **Graydon**: can we change check target to not do tidy...
  * **Elliott**: can we do it at the end?
  * **Graydon**: just add a new target; check-and-precommit or whatever
  * **Eric**: yeah, make precommit makes sense
  * **Graydon**: make commit!
  * **Tim**: yeah, make commit
  * **Paul**: could commit automatically
  * **Graydon**: some workflow to do there, but we can do that

## Updates

  * **Patrick**: coherence is about to land
  * **Eric**: checking or enforcement?
  * **Patrick**: enforcement.
  * **Patrick**: and switching typecheck method checking to use it, so you don't have to import impls anymore
  * **Graydon**: woo!
  * **Patrick**: impl map still built in resolve but not used anymore, could probably just go delete that
  * **Graydon**: you could do it after you land, not a big deal
  * **Patrick**: will need a snapshot. currently you can name impls, names don't do anything anymore, but needed for compat
  * **Patrick**: ability to add multiple traits needs to be added too
  * **Patrick**: a little more stuff needs to be done
  * **Patrick**: do have to import traits if you have extension methods
  * **Graydon**: explain?
  * **Patrick**: if I imported a `to_json` trait and implement `to_json` on ints, if you call `to_json` you need the trait in scope, because some other crate could define a `to_json` trait
  * **Niko**: if you call methods on type where the type is defined, you don't need to
  * **Patrick**: yeah, I call those "inherent" methods, as contrasted with "extension methods" (or "post-hoc")
  * **Patrick**: Haskell has same rule
  * **Dave**: it does? never knew that!
  * **Patrick**: that's what `import qualified` is for
  * **Graydon**: what's the failure mode?
  * **Patrick**: just says "I can't find a method for that" -- in future, could give suggestions
