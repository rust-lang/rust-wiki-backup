## Attending
Dave, Brian, Niko, Patrick, Marijn

## Triage

* Graydon: one thing: thinking of having organized periods of bug triage during the weeks; bug list constantly growing, can't keep track (dups, important, etc)
* Graydon: etc)
* Graydon: wanted to spend some of one of the days this week doing triage; anyone wants to join?
* Brian: propose we do?
* Graydon: try to get control! get in habit of spending some time every week addressing issues, maybe as a group, maybe individually
* Niko: do yourself, divide it up, take turns?
* Marijn: one thing: policy on when to close stuff that looks far out or not going to happen; starting to clutter up list; reorganize into bigger bugs? hard to scan because of junk

## Bug tracking

* Marijn: one issue I see is we have no person w/ final word on "not going to happen" -- should we make a list and cycle it on mailing list? but maybe too much process for bugs popping up like mushrooms
* Patrick: priorities can help
* Marijn: github has no support for that
* Graydon: yeah we'd have to use tags
* Niko: any way to see things that are *not* tagged?
* Graydon: no, no set operations on tags; really wish they had something more sophisticated
* Graydon: I think we can probably make do with brute force so long as we have some action we can take on each bug to make it get better
* Graydon: don't think we can just stare at bugs and wish them to go away
* Graydon: maybe a prioritization period, after which have to prioritize
* Niko: I'm ok with being more aggressive about closing; can always reopen
* Graydon: priority number levels don't clarify much; prefer to mark as a specific intention (e.g. rfc tag)
* Dave: closing a bug seems ok if it's not a priority for forseeable future; can always reopen later when it becomes a priority
* Graydon: sure; maybe if 2 or more people want to close it, close it; maybe one person can nominate for close
* Dave: so a "close" tag to nominate, then we can scan and decide either to close or untag
* Graydon: yeah, a concrete step to push through an explicit lifecycle
* Niko: maybe an additional "maybe" tag to close with, to avoid offending
* Patrick: milestones, "wanted but not blocking anything"
* Marijn: but then closed bugs show up in milestones
* Niko: class of things we want to do but no one's had time to do yet (e.g., distinguishing mutable and read-only)
* Marijn: also a lot of stuff that is bugs that have to be done at some point; maybe now and then we should reserve a week for bug work instead of new feature week
* Graydon: agree
* Niko: yeah, should be some mix
* Dave: we should nominate bug weeks; makes a lot of sense
* Graydon: a while ago I suggested process before release doing bug work; there was some objection, I think because it was very rigid
* Graydon: but in a broad sense, make sure to do these before releases?
* Brian: let's try it
* Niko: yeah

## Non-exhaustive alts

* Graydon: anything else?
* Marijn: anyone object to me implementing a way to mark an alt as non-exhaustive?
* Niko: I don't know if I object, but I object to it being used widely in the compiler; we shouldn't use _ so much; should enumerate the cases that we don't handle so that we can tell what we're missing when we add new cases
* Graydon: I proposed alt check form
* Niko: IIRC that was unchecked
* Graydon: yeah, just non-exhaustive
* Marijn: so Niko, you prefer alts to be exhaustive?
* Niko: I just don't want to use non-exhaustive in the compiler
* Marijn: agreed; also tons of stuff where you just want to handle one case
* Niko: sure, and in those cases non-exhaustive makes sense

## Regions and assignability

* Niko: I've been tossing around more region stuff; plan to send out revised proposal in a little bit, trying to think about handling region pointers in types with minimal notation overhead; nothing concrete to talk about just yet
* Niko: also: implemented assignability check, haven't yet decided if I think it's high-impact; most common pattern is fold-left pattern; leaning towards maybe not making assignable class and tracking immutability better
* Graydon: ok; doesn't entirely feel like we're hitting the problem case that often right now; good to solve but not blocking us in the meantime
* Niko: y, quite a corner case

## C-Rust functions

* Patrick: Brian, could you explain how C-Rust functions work?
* Brian: declared like normal functions but with "crust"; not callable, opaque pointer; follows cdecl ABI
* Brian: when you call them from C:
  * they bundle up the arguments into a struct
  * call a simple shim function that takes args and switches back to Rust stack
  * then that calls a regular Rust ABI function that actually implements the body of the function
  * instead of commandeering scheduler's stack, always has one big stack available, and leases it to whoever's requesting it for C
* Brian: can do almost anything in Rust stack going back and forth; can still yield when calling back into Rust, but can't fail
* Brian: but in yield checks, some things can fail, so tasks can't be killed while they're calling into C; the death is delayed till when they come back (can code around this with a monitor task that gets notified)
* Niko: could add a flag somewhere
* Brian: event loops will have to do that
* Dave: didn't get one thing: will it allocate another C stack if there are none available?
* Brian: yes, but doesn't ever happen unless a task has recursed back into Rust and yielded
* Niko: a pool?
* Brian: if there was an extra one created, it gets freed
* Dave: so if you recursively cross Rust/C boundary and yield, you'll need an unbounded number of C stacks; answer is "don't do that"
* Brian: hm, but we need that for event loops!
* Niko: no
* Brian: good point, but we can address this

## Wrap-up

* Dave: we can schedule a bug week on IRC or the list
* Marijn: so we'll add a tag that's candidate for closing, and periodic searches through these candidates
* Graydon: I'll make a proposal on ML about redoing the tags; they're not really capturing the right dimensions
* Graydon: worst case, we can migrate to a different bug tracker, but maybe they'll fix it up
* Niko: do people think it's important to label with stuff like "trans" and "rt"?
* Graydon: not so much
* Niko: I don't really think it's helpful
* Graydon: only sort of one dimension to track, and there's lots of other dimensions to track that are much more important
* Marijn: probably assigning them, even if you just unilaterally assign, probably helps; makes someone the owner
* Graydon: yeah
* Niko: that seems reasonable
* Niko: policy: all bugs not closed should be assigned?
* Brian: assign to rustbot :)
* Graydon: unfortunately we had to invent "unassigned" tag
* Brian: get rid of it -- too hard to maintain
* Graydon: agree
* Graydon: ok, I'll attack this as well as the rfc proposal, since I didn't see disagreements to my email
* Niko: I don't think I saw it; I'll take a look; go ahead and send it out and I'll read it
