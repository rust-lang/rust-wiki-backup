Meeting notes, 2012/01/17
## Attending:
Brian, Niko, Marijn, Graydon, Dave, Patrick, Tim
## Vidyo Joy
* M: let's try G+ hangout next week

## Release status
* M: discussion points?
* G: not especially. I can explain what I'm blocked on, but I'm getting through it. getting ref manual into a presentable state. not far from done; not sure how thorough needed to be for release
* M: not important to be thorough since we're going to change
* G: just don't want to be full of lies, and would be good for grammar not to be ambiguous
* N: you're manually converting parser code to grammar?
* G: no, converting what I think the grammar is into a grammar. just writing it down
* M: there are some pretty scary things in the parser. not sure how verifiable it is
* G: I think we have to try
* D: I agree, but 0.1 not necessary
* G: I agree
* N: we don't wanna be C++
* G: it's too much of a rathole to say only ref impl can parse this thing
* D: that's not controversial
* N: I need to finish purging out function types; not 100% done but code is there; waiting for snapshot to purge old stuff out
* M: tutorial pretty much ready to go; only installer section left
* G: we need a little work on "getting started" on wiki and in tutorial
* M: status of windows installer? how are we going to release on unices?
* G: unix: tarball; windows: installer builds as part of `make dist` -- spits out an .exe
* M: is it available online? would be nice not to have to build
* G: no, but I'll wire up the builtbots to snapshot and upload to release directory on tags; I don't want any manual steps in this process
* G: probably won't have any other OS packages developed at this point; but this may draw attention
* G: I'm seeing 0.1 as more of a "statement of intent" to the world than a finished project; I expect we'll get a bunch of attention, along the lines of "hey you're not a good software package yet; you need to do X Y Z" -- 0.2 and 0.3 will probably end up being lots of structural/project stuff
* M: there was talk about this week last week :) are we ready?
* G: I don't see much that needs a whole week of effort; could still happen by Friday
* G: anyone else feel like there's something right now that is absolutely blocking?
* (heads shake no)
* G: I pasted a "release notes" etherpad, which is typically self-deprecating and negative
* B: say some awesome things!
* N: yes, we have quite a lot to mention
* G: I think the level is essentially feature-complete, at least at an alpha level
* N: should we look at the bug list?
* M: anyone have too much on their plate? I'm out of blockers & can take something
* G: I kind of am
* M: I can take Makefiles

## Bug review

### 1427
* G: how far are we on this?
* M: Tim has been working on it, but not finished
* G: I'm okay to be a little ahead of the curve and remove from the docs
* M: I'm still not sure this will work out, so I would be cautious there; it'll bite anyone who tries to write a program by the docs
* P: let's check with Tim
* B: we'll get it done
* (Tim joins)
* N: Tim: what's status?
* T: more annoying than expected, but should be ready to check in today
* P: if so I'll try to commit comma thing and we can do one snapshot for both

### 1428
* N: where are we on this?
* P: been trying to do snapshots
* B: they're done
* P: okay, then I can run my Perl scripts
* D (in his head): Perl FTW

### 1486
* B: no way rustdoc is gonna be ready this week

### 1483
* N: we can go a little stronger on this now
* G: yeah that's easy
* N: just purge it, right?
* M: yeah
* N: did you add options separated by comma?
* P: no, not yet. shouldn't be hard
* N: just snapshot is hard
* M: I can also do that tomorrow. just a lot of regexping, if Patrick is busy
* P: I have a Perl script. I've tested it. I'll do it today, including commas

### 1381
* M: shall I take this?
* G: yeah sure!
* G: the way cargo is set up: essentially 2 contradictory goals: 1) decentralized; none of us is a choke point on adding packages; people can mature packages independently 2) encourage people to use a central service, flat namespace, searchability
* G: we wound up wiring in 2 points of indirection: list of sources in home directory, populated by default; points source list to files; each source can have lists of package locators
* G: in practice, every person who wanted a package made their own source
* M: where is the main source located?
* G: ATM, github/graydon/cargo-central -- should be moved
* G: ideally, JSON file will have descriptions of most packages most people will want to use most of the time
* D: directory-local vs site?
* G: orthogonal question; it's about who maintains list of packages
* P: can it just be something that someone can manage on github and accept pull requests?
* G: yes, that's the goal
* P: would Elly be interested?
* G: yeah, sure!
* G: default source should point to github/mozilla/cargo-central
* P: yeah
* G: I'll promote to mozilla now, and make notes in the bug about what's needed in order to make it public
* G: what I want is that whenever someone has a kneejerk thought about creating a cargo package they submit it to cargo-central
* P: should be pull request-based, so we can merge with one click so we don't go crazy
* M: is a JSON file the best way to do this? is a custom web form system better?
* G: reason we did Git as backend was so there'd be no infrastructure difficulty to set up your own
* D: there should be a Github API we can use, right?
* G: there is. we could authorize a script to fork and commit
* N: <<alternative approach, Dave missed>>
* M: <<something else, Dave missed>> -- but can't do auto-merge on Github, that's probably a problem
* G: solving that set of problems is all I wanted to get at
* B: should we encourage people on IRC to move their stuff onto cargo-central?
* M: I think we can do that unilaterally
* B: I think there are problems with the same packages in multiple sources right now
* G: let's nip that in the bud
* P: let's put it in the IRC topic right now
* G: yes, gotta be extremely obvious
* G: just moved cargo-central to mozilla/cargo-central. now a lot of people can administer it
### 1011
* B: only thing is to add installation PATH to system PATH in installer
* B: I'll do that today and do some testing on the windows installer

### 427
* G: Marijn: you're currently processing tutorial using node
* G: what I want to happen is that whenever we push, documents rebuilt and repushed
* G: node is not installed on the buildbots
* G: I could install it, but I was wondering if it's possible to switch that processing to a Python script
* M: we'd lose highlighting; there's custom filtering to verify they're up to date
* M: would installing node be a big problem?
* G: bunch of work, but it's ok
* M: not strictly necessary to be automated by 0.1
* G: no, not strictly. just want to get to it
* G: similarly I'd like manual formatting and tutorial formatting to match
* D: pygments is a pain to customize and install
* M: and I'm using context-aware keyword awareness with codemirror, so I don't want to lose that
* N: do we have any checks that the examples parse and run?
* G: not currently; long-term would be good
* M: you have to be careful to have some extra magic to avoid long examples

### Miscellaneous
* G: Marijn: did you see the markdown file with remaining work items?
* M: no
* G: there's a bunch of stuff left to convert from the texinfo
* M: I'll just pick up tomorrow morning (my morning) and finish what's empty
* G: what I've added to that document is: it's written in dialect of markdown understood by pandoc; slightly different from github dialect and dialect understood by your script
* G: it understands footnotes; uses slightly different syntax for marked syntax blocks (triple-back-quoted whatever)
* G: for syntax blocks: I have a script for ones marked as EBNF; pulled out by grammar analyzer; otherwise left in place
* G: if you'd like to add EBNF rules for grammar productions you're confident about, that'd be cool
