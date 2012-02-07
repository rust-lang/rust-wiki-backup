## Attending

Brian, Tim, Patrick, Niko, Dave, Marijn, Graydon

## Bind

* Niko: will put in small change that makes `bind` unnecessary but doesn't change anything
* Brian: I question whether that's worth it, since it's just syntactic, and the syntax isn't the problem
* Patrick: yes, syntax is the problem
* Marijn: we won't get stack-allocated bound stuff anymore?
* Niko: could make it work; the more painful thing is supporting arbitrary expressions; need a special case for each one; for partially evaluated expressions it takes more work; could make stack-allocated based on simple inference, based on the expected type

## Monomorphization

* Patrick: how's monomorphization going?
* Niko: Marijn's working on it; I'm working on serializing the AST instead of pretty-printing; avoid reconstructing everything we already have; I was tempted to write a utility based on `#ast` but I'm just gonna push through; open to alternative approaches
* Patrick: fine with me, pretty-printing is strictly worse, just did it at first for expedience
* Niko: not sure it will even work
* Patrick: yeah
* Niko: last step before trans; really just one main crate needed
* Marijn: resolving stuff in pretty-printed snippets seems just about impossible anyway
* Niko: and we want proper line numbers
* Graydon: agreed
* Marijn: I've started on monomorphization; easy once we have cross-crate inlining
* Niko: great

## Proposal process

* Niko: should we discuss RFC process?
* Patrick: I'm in agreement with both Marijn and Niko; just best to use judgment
* Niko: for something uncontroversial, no need for process, e.g. changing the precedence of `as`
* Dave: just want confidence of consensus; use your judgment as to whether you have it
* Niko: there's a gray area between obvious vs. needs-more-communication
* Tim: my preference is to err on the side of communication
* Marijn: with operator overloading, it's clear there was a failure of communication, so some process is needed
* Niko: IMO it just wasn't specific enough to know what we were discussing
* Patrick: agree
* Graydon: I'm actually okay with formal process, but the RFC tag currently isn't getting any attention in the issue tracker, and there seem to be bunches of RFC tags that aren't even relevant; we should go through them
* Graydon: I think we should have a formal "show of hands" way to tell whether we have consensus
* Niko: but what's a good number? everyone?
* Dave: must be everyone; unanimity is important; that doesn't mean everyone always gets their way, but everyone needs to be on board
* Tim: agree
* Graydon: want clear guidelines for what is contentious (e.g., if it changes the AST or needs an updated entry in the reference manual)
* Niko: OK, we'll write some up; fall-back is "ask"
* Graydon: mailing list is fine; don't have to wait for meeting if you're trying to get go-ahead
* Niko: just need to make that clear in the email ("I'm gonna push, so...")
* Graydon: I'll write this up
* Tim: can I leave off "+1" emails?
* Niko: as long as it's clear in the email that agreement is needed
* Dave: you can also say if you're uncomfortable and want someone to wait but aren't quite sure why yet; don't have to have a complete story to reply
* Niko: can also say "go ahead with this part, but I'm uncomfortable with this other part"
* Marijn: objecting after the fact is OK too
* Niko: I agree
* Graydon: true
* Niko: "go ahead but I reserve the right to complain later" is a valid response

## Regions

* Patrick: the unsafe pointer patch that Marijn landed seems like a feature request for regions
* Graydon: yes
* Niko: I'm cleaning up the regions proposal; I'll send it out and we can discuss on the list
* Marijn: GC would also solve it, but regions is interesting
* Graydon: really hesitant to say GC "solves" it; adds a new cost center
* Patrick: understood
* Marijn: cost was clearly RC traffic in this case; pretty confident it would fix this particular issue; but anyway we should experiment
* Graydon: I agree
* Niko: think we should experiment with GC, perhaps we should agree on some benchmarks
* Patrick: well, a good GC takes a long time, and you don't want to discount it based on the first stupid mark-sweep algorithm you write
* Patrick: I sympathize that GC is hard
* Dave: my feeling is that our RC is technical debt and we'll need a real GC
* Niko: properly implementing RC is gonna cost
* Graydon: not saying I don't like GC; heck, RC *is* a kind of GC; we have RC + a broken cycle collector; but others are hard, and every technique we choose will cost
* Marijn: or do regions alleviate the need for a proper GC?
* Niko: I doubt our proposal is strong enough to type everything you'd need
* Patrick: maybe; we can discuss offline
* Graydon: also, it's worth considering unsafe implementations that can be confined to libraries; e.g., pin a constant to task lifetime and everyone borrows references; we already have non-owning pointers in the language
* Marijn: our safe references are not first-class and wouldn't work; I think Niko's proposal of first-class reference could be really valuable
* Graydon: just want to keep in mind unsafety that can be sequestered to library code
* Niko: I strongly agree with that last part; and task-local data is a good idea; but Marijn is right that we can make these particular patterns better with first-class references

## FFI

* Brian: one last thing: I want to implement callbacks from C to Rust
* Marijn: what you outlined on IRC sounds really good; I'm definitely in favor
* Graydon: me too; I'll check out the RFC
