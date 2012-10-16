## Attending

brson, nmatsakis, pcwalton, dherman, jesse, graydon, tjc

## 0.4 Release Coverage

  * http://www.reddit.com/r/programming/comments/11j38z/mozilla_and_the_rust_team_announce_version_04_of/
  * http://news.ycombinator.com/item?id=4657040

## Overall priorities

  * **Graydon**: perf or other stuff?
  * **Brian**: perf will haunt us if we don't address it; will haunt us otherwise
  * **Niko**: specifically build cycle, or benchmarks, or both?
  * **Graydon**: both
  * **Graydon**: much more concerned about benchmarks, but build cycle affects day-to-day experience of us and of users
  * **Patrick**: we can fix some of the C++ part by not building parts of clang we don't need
  * **Niko**: for reputation, build time is probably more important than any benchmark
  * **Niko**: we do okay on benchmarks, right?
  * **Patrick**: not on shootout, but I personally don't care about that as much; I want to finish the language; as long as we have a roadmap to getting the perf we need
  * **Patrick**: I feel like perf work is done better when you have features implemented, following knuth's advice
  * **Graydon**: not trying to argue that position; just wanted to gauge people's feelings
  * **Niko**: wasn't really on my radar
  * **Patrick**: I have a reasonable, ~70% guess what issues are already; fixing to use fast ISel would take maybe a week or two if we're lucky
  * **Patrick**: our perf is fine for real-world code, but not optimized for shootout; outparams are an issue
  * **Graydon**: stack, prolog costs...
  * **Patrick**: lot of it is straightforward
  * **Niko**: significantly improving build time would help all our productivity
  * **Brian**: I'm self-conscious about build times b/c a lot of it is my fault, stuff I did; if we could get it down... I did a quick test this weekend where I found we do so many extra builds... I think we could get make check stage1 down to 10 minutes
  * **Graydon**: if we're talking about reorganizing builds generally, that's on my list of important topics
  * **Niko**: fixing fast isel, handling windows exceptions... these are inter-related
  * **Graydon**: yeah, I'm into the windows one

## 0.5 priorities

  * **Graydon**: see https://github.com/mozilla/rust/wiki/Note-0.5-priorities
  * **Graydon**: if we only have a couple months, this list is way too long; want 2 - 3 months, not 4
  * **Dave**: everyone starts disappearing in mid-December
  * **Niko**: let's shoot for December 15th
  * **Graydon**: if everyone can pick a single discrete task, bit of a challenge, maximum bang for buck, what will it be?

### Dave

  * **Dave**: I kind of agree we should not get too obsessed with performance, we should push for feature completion, but if we can get low-hanging fruit and productivity boosts I agree with that
  * **Dave**: My only other vague opinion is that library design/organization are going to become very important and we probably need some internal effort to organize them and provide overall structure.  But that is more of a 1.0 priority.
  * **Graydon**: I hear that as a vote for traits

### Patrick

  * **Patrick**: I started on fixing traits... that was deliberate; didn't want it to...
  * **Graydon**: there were still a couple things that came in the tail end
  * **Patrick**: not important
  * **Patrick**: default methods, trait inheritance mostly work instead of mostly-don't-work, but still "mostly" -- default methods don't work with generics; deriving requires some design work; niko and I probably need to hash that out
  * **Patrick**: I got &trait working, ~trait doesn't
  * **Graydon**: fantastic!
  * **Niko**: awesome
  * **Patrick**: yeah, already half-done so might as well finish
  * **Patrick**: other thing is: remove obsolete features (modes, exports, structural records, ...)
  * **Graydon**: you can keep doing that till very last day
  * **Patrick**: not prioritizing removing them from rustc, just from libs
  * **Niko**: that makes total sense. something like arrow and swap that we can drop entirely, then great
  * **Graydon**: structural records we should probably get out of libs asap
  * **Graydon**: does &trait occasionally inline if it knows the target?
  * **Patrick**: haven't verified this but I assume so
  * **Graydon**: that's key; not going through malloc/alloca, supposed to track that

### Brian

  * **Brian**: my priorities: Windows unwinding, build system & cargo; not only Rust but Servo also has a terrible build system
  * **Graydon**: oh no, Servo has it too?
  * **Brian**: in some places literally copied from Rust build system
  * **Graydon**: oohhhh nooooo!
  * **Niko**: there's serious design questions here. I don't know what I think, but maybe you do
  * **Brian**: no, my general approach has been hacking in features by need but haven't really designed
  * **Graydon**: there's merit to that, but there's merit to design up front too
  * **Niko**: is there a best-of-breed package manager we should look to?
  * **Brian**: nvm
  * **Patrick**: npm seems widely popular
  * **Dave**: npm does have some criticisms, but it seems like a step forward
  * **Patrick**: need to handle not just build but building external C dependencies
  * **Dave**: feel like someone should really study existing systems well
  * **Patrick**: almost feel like this could be someone in the community, who wants to contribute to Rust but doesn't want to get dirty with the compiler
  * **Niko**: we need the cargo boss
  * **Graydon**: this is tightly related to concept of versioning, and versioning is a language-level concept in this language; I feel like this is more subtle than we can do outside the compiler. having compiler agnostic about packages is a mistake
  * **Dave**: path is littered with bodies of those who've tried to solve language-level versioning; best thing to aim for is an incremental improvement
  * **Graydon**: that's all I want. allow multiple versions to coexist; can correlate generated code with source; want to integrate with good version control
  * **Niko**: worries we're in "perfect enemy of the good" territory. brian: is there incremental stuff you can do in this cycle?
  * **Brian**: certainly; servo has a few needs that could be solved w/o any deep thinking. running build commands, managing git repos
  * **Patrick**: as long as we can do that; ability to build C code and call out to external makefile; the other thing I'd like is static linking
  * **Patrick**: right now I can't build rust programs that survive more than a day; update compiler and all apps that depend on them die
  * **Graydon**: these topics are related. I gather that hacking around is a way of solving the problem, but it's a way of deferring solving the problem.
  * **Dave**: we don't know where we're headed. we're going to have to fix that at some point.
  * **Graydon**: yes. I'm willing to take some time thinking about this.
  * **Dave**: people will depend on it fast
  * **Brian**: well Servo is the largest Rust codebase; solving its needs will help us answer these design questions
  * **Graydon**: don't disagree, but looking at Servo's needs means thinking about these problems; static linking means "let's not solve this problem"
  * **Patrick**: not proposing that. Servo's needs *are* static linking
  * **Graydon**: okay.
  * **Dave**: I think we're seeing violent agreement; we need to address these problems, the question that remains is just how much we attack in 0.5 cycle
  * **Graydon**: just want to make sure we don't end up drifting down path of being more and more dependent on make/bourne shell, etc

### Niko

  * **Niko**: focus on mopping up unfinished portions of region system; in particular completing function-type transition
  * **Brian**: unsoundness of unique functions is terrible right now; in Servo we capture pipes all the time and it's really tricky
  * **Niko**: yeah, that's one example of the kind of problems we have. kinda know what we need to do, just have to do it
  * **Niko**: related to: better treatment of extern functions, other stuff listed on page
  * **Niko**: I think that'll take 2 months
  * **Graydon**: sounds like a long sequence of small, well-scoped issues. makes sense

### Tim

  * **Tim**: dependency generation; almost everything else had been written down already.
  * **Tim**: was part of my long-term plan to do incremental compilation
  * **Tim**: useful for build times, potentially also for build system
  * **Tim**: labelled break & continue is also assigned to me; since I haven't had a major project in a while, I'm open to doing anything, but dep generation was the big project I had in mind
  * **Graydon**: I'm a little split on that
  * **Graydon**: labelled break & continue is blocking Henri, right?
  * **Brian**: yes
  * **Patrick**: is it still?
  * **Brian**: yeah, he's expecting us to deliver it to us. working on it w/o, but he needs it
  * **Niko**: simple version is not too hard. one that doesn't integrate with for-loop
  * **Tim**: I didn't understand the liveness code. could sit down with you
  * **Patrick**: just want it for loop, not sure ever want it for for
  * **Niko**: I disagree. could sit for a while but I think it *should* work
  * **Graydon**: can sit till other things are working
  * **Graydon**: I'm really interested in dep generation, but: what I'm hearing but correct me if I'm wrong... short-term build issues and long-term build issues
  * **Graydon**: long-term: formalizing dependencies, formalizing versioning, understanding what that means
  * **Graydon**: short-term: getting off make, getting static linking, getting sub-makes working
  * **Graydon**: short-term could be 0.5; long-term all we can do is talk. that's totally cool and I wanna do it, but that's not a coding task
  * **Graydon**: can possibly involve 1 - 2 days a week, or time in person, or video conferences...
  * **Dave**: sounds like a good idea
  * **Graydon**: bunch of stuff to talk through in this space
  * **Graydon**: involves work that can happen in parallel with short-term work and getting us off of make
  * **Brian**: I'd like that
  * **Graydon**: RE: coding work for this cycle, Tim is labelled break & continue good?
  * **Tim**: yeah, don't think it should take two months, can look at other stuff too
  * **Niko**: one thing I saw on list: Re: modes: last vestige that's user-facing is pattern-bindings. adding in some legacy pattern-bindings thing might be right way fwd
  * **Patrick**: I think it is. there's a warning currently
  * **Niko**: there is, but no way to say "change the default" to what we want it to be
  * **Patrick**: in my experience #1 source of borrow-check errors
  * **Niko**: agree, you don't realize you're borrowing
  * **Niko**: legacy bindings is fairly straightforward and maybe we should do that
  * **Graydon**: Tim, is that something you could pick up as well?
  * **Tim**: sure. to summarize, this would mean...?
  * **Graydon**: issue #3271
  * **Patrick**: introduce legacy patterns thing, by default copy
  * **Tim**: clean up libs, or are they already not using legacy patterns?
  * **Niko**: I would start by putting in the legacy switch in libs, then we can work on getting rid of it

### Graydon

  * **Graydon**: we have buildbots running
  * **Graydon**: my biggest concern is build automation stuff; wanna get us to better build automation. new stuff has wonderful flexibility: in particular can have it schedule additional tasks periodically or dependent builds (fast build, slow build, even slower build, multiple types of machines), unlimited quantities of hardware
  * **Graydon**: if I have any time leftover after that, or in parallel (while waiting)... start on new driver?
  * **Niko**: how hard would it be to get nightly downloadable builds? that would be really awesome
  * **Brian**: if we did, would we encourage people to use that build and not the release?
  * **Niko**: we encourage people to use git now
  * **Patrick**: that was between 0.3 and 0.4
  * **Patrick**: diff between 0.4 and 0.5 better not be as big
  * **Graydon**: I think 0.4 was our biggest disruption
  * **Niko**: people will still hit bugs, etc
  * **Niko**: if we could just say "download nightly build from master" then they wouldn't worry about build times so much
  * **Dave**: would also stave off some of the complaints about build times
  * **Graydon**: good point. I have seven i7's sitting next to me so I don't think about build time as much :)
  * **Brian**: they still have to build, don't they?
  * **Niko**: I'm talking about binaries
  * **Graydon**: it would definitely be more work
  * **Graydon**: part of doing versioning would be being able to distinguish source and binary at tool level; don't always want cargo to work by downloading all source and rebuilding. pretty crude.
  * **Patrick**: npm doesn't have to deal with that, yeah.
  * **Graydon**: and we're on OSes that have package managers, so should be able to integrate e.g. with dpkg
  * **Patrick**: distributing binaries on Linux is another concern; Linux systems don't like that
  * **Niko**: hadn't really thought about the complications in detail, but I still think it'd be good to do if we can do it
  * **Graydon**: at least want to switch over automation system
  * **Graydon**: wanted to run by people: if I have residual effort, where to spend it: new driver code vs condition handling system
  * **Patrick**: latter probably won't take too long, I think
  * **Graydon**: no, shouldn't take too long. can get a sketch version implemented in a day. can probably get a debug version working within a week
  * **Graydon**: contributes to library design question. libs are furiously annoying to write if you want them to handle partial failure (as opposed to complete failure)
  * **Niko**: reminds me of one thing: when is John Clements coming?
  * **Dave**: Jan 7
  * **Niko**: speaking of result monad, would be nice to have a macro for some result handling, would be nice to import macros, but that's a long way off, never mind
  * **Niko**: anyway, I agree it would be good to sketch condition system out
  * **Graydon**: ok. anyone think it's more important for me to work on new build driver?
  * **Dave**: I'd vote for condition system so we can move towards libs
  * **Tim**: I'd also vote condition system; have been burned by this enough

### Jesse

  * **Graydon**: Jesse, are you going to make the fuzzer solve all our problems?
  * **Jesse**: no. :)

### Meeting time

  * **Tim**: I've been wondering for a while if we could change meeting later?
  * **Patrick**: I like at least one day a week being forced to get up early
  * **Graydon**: me too!
  * **Tim**: for me I do it at home and get to work at 11, or sit in traffic for an hour

### Minor priorities for 0.5

  * **Brian**: our 0.5 milestone list has hundreds of open items
  * **Tim**: things can be punted to 0.6
  * **Dave**: so we've done major triage, but now there's minor triage left to do
  * **Niko**: seems like an issues system problem
  * **Patrick**: unavoidable. happens in all software projects
  * **Tim**: just need to go through regularly and reprioritize
  * **Brian**: I can go through and start punting things with some confidence
  * **Dave**: maybe just keep a list of things you're not sure about
  * **Tim**: ICE's I tend to put as higher priorities since they're user-facing and embarrassing
  * **Patrick**: still wanna land my patch that has a 1% chance of saying "Infernal Compiler Error"
  * **Tim**: perfect for Halloween
  * **Patrick**: or Friday the 13th
  * **Graydon**: gonna set target of release for Tuesday Dec. 18th
  * **Graydon**: and congratulations to all for the 0.4 release!! very excited about the state of things
