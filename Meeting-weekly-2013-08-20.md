# Agenda

- cycle time
- some interns are leaving

# Attending

graydon, nmatsakis, kmc, jack, felix, sully, azita, toddaaro, lbergstrom, ecr, brson, tjc, dherman, bblum

# Cycle time

- G: We need to focus on cycle time. Everything we measure seems to be too slow. It's a real drag on the progress of the project. As a consequence, work sits in the queue, rots, and is demoralizing.
- Azita: More hardware won't help?
- G: No. We're using the best hardware we can but...
- N: Are there gross profiles available?
- G: In the test suite, we spend our time almost entirely in metadata
- N: Is there a breakdown showing e.g. stage0, stage1, stage2, tests, and perhaps a bit more detail?
- G: Those numbes are available, this link shows the primary breakdown of the worst offenders:
http://huonw.github.io/isrustfastyet/buildbot/all
- G: You can see that the test suite and compiler times are completely different depending on what block you are doing
- G: The all builds are limited by the amount of time spent building rustc, winds up doing 9 builds, each build takes ~3.5 minutes, and it takes about an hour to do that 9 times + libraries
- G: Bottom line you can see that by far the worst offender is valgrind
- G: Two things going on: new runtime has technical debt surrouding I/O and process launching, we're doing the worst possible thing in several different dimensions, something to do with locking on waitpid, which is being worked around by creating a lot of threads, which leads to valgrind serializing things, and we're also using multiple threads for stdin, stdout, etc etc
- G: Subsequently another issue is that just compiling the test suite itself is slow, 45 minutes to run the test suite on an optimized build
- G: 90% of that is spent reading metadata
- G: If you look, the actual content of the test suite -- how long the tests take to run -- are basically zero, it's mostly just reading and re-reading libstd
- G: I'm not triyng to chastise just suggest that it's time to prioritize this work
- G: A lot of bugs that I don't personally know how to fix
- N: Are these labeled or otherwise easily identifiable?
- G: I'll try to put up the ones I think are the biggest problems
- sully: I have at least a gimmick suggestion, we have the "fast" mode that we use on windows. We could conceivably run just the xfail-fast tests normally and use fast for the rest.
- G: True. We can also turn off some testing. There are various ways to rejigger the test suite. 
- pcwalton: Can we run valgrind only post commit rather than pre commit?
- G: We did that before but everything just broke and we stopped paying attention to it.
- pcwalton: We were making larger changes at the time?
- G: This is a direction I don't want the meeting to go.
- pcwalton: Every other project at mozilla does it after the fact?
- G: But every other project doesn't spend 90% of their time reading metadata...I'm tired of the answer being less testing.
- pcwalton: I feel like I don't want you me to do. Should I stop working on the backwards compat features and work on metadata instead?
- G: Yes, I think we should stop working on features and focus on performance
- pcwalton: There are multiple kinds of technical debt, backwards compat language features are debt too
- G: I understand but right now people just give up trying to land things
- nmatsakis: How much work is it to improve the metadata results? Getting the cycle time down would certainly be a multiplier
- pcwalton: Gecko's test suite takes longer than ours...?
- G: They close their tree regularly, as well, we've been there.
- brson: Do we know what it will take to fix metadata?
- pcwalton: Yes, I know what it takes, but I don't know that it will be a massive win. We'll still have to read some metadata.
- G: I just mailed out an e-mail with profile. This not the biggest bottleneck for straightline compilation, but for the testsuite it is.
- G: The other big thing is that we do not seem to be scheduling running subprocesses and reading the results properly. We are very pessimistic.
- brson: I understand this is a complex issue. If we disabled the all and valgrind bots how much time would we save?
- G: That would cut the time from 2 hours to 1 hour.
- brson: So I'm very sympathetic that we need to fix the root problems. But if we can go twice as fast just by turning off those bots, maybe we should do that in the short term.
- G: I'm willing to do so but I want to have other people doing the work to get those other issues simultaneously.
- brson: OK, there's metadata and process spawning. What else?
- G: A ton of codegen stuff we can do.
- G: Nobody else has even added a codegen test since I added the instrumentation and all those tests are very bad compared to clang.
- N: Does it have to be pcwalton who does the metadata stuff?
- pcwalton: It sounds like I need to stop everything and do it, so I can do it.
- G: I'm willing to try but it's in areas I do not know.
- pcwalton: It's fine, I'll do it.
- azita: How much work are we talking about?
- pcwalton: I think around a month.
- pnkfelix: Patrick mentioned being worried about backwards compatibility issues -- is this something we should broadcast to the community to warn people not to do too much?
- pnkfelix: We might want to discourage r+'ing things
- pcwalton: We can close the tree, but otherwise stuff will go in
- pnkfelix: We can take people off the approved list of r+
- pcwalton: We can require approval of every change ("approvers" vs "reviewers")
- pcwalton: I don't necessarily care if there's a long list of approved changes in bors
- dherman: Can you guys back up a sec? What exactly is the problem? What's this about people's commits being a problem?
- ... explanation proceeds ...
- G: post commits mean that you run tests on a set of commits, but you don't know which commit is at fault. Our experience with this was that errors would accumulate and we wouldn't know how many errors had snuck in in the meantime.
- dherman: I guess my last question is, assuming we stick with the pre-commit testing, is this even a solvable problem? If you just keep growing your testsuite, won't we always return to this situation?
- pnkfelix: I'm not sure if Niko properly represented the situation. Picture that I get from isrustfastyet.com is that bootstrap compilation / test-suite is 50-50.
- graydon: Basically true.
- dherman: So certainly working on compile time would be a multiplier
- dherman: I don't really understand the scale of the test suite
- graydon: I don't think the test suite is going to grow at pace with Moore's law
- graydon: As the compiler matures, I would expect it to accelerate
- graydon: We get worse because our technical debt builds up, not the size of the test suite that's the problem
- pcwalton: Our compile times have been going in the right direction lately
- dherman: My preference would be that we don't completely stop and shift gears entirely, but it seems to me that we could divide our time between trying to bend the curve, balance it with continued feature work. The idea of stopping everything is scary but this does sound like a real problem that needs attention. Is that possible? Unrealistic?
- dherman: Patrick I don't think you need to drop everything for a month.
- pcwalton: Problem is that the resolve stuff is my domain. It does perhaps make sense for me to drop and focus.
- graydon: You're actually much more spread out, I've been basically doing dev-ops for the past month. Is there any way you could spend some time telling me how to fix? I feel like throwing it in your lap over and over again is rude
- dherman: and it's good to throw things 
- graydon: I feel the same way about the move issues that are blocked on Niko
- pcwalton: Are you talking about avoiding zeroing out?
- graydon: Also avoiding calling drop glue
- nmatsakis: That specific issue I can take a day or two to focus on and delegate as needed
- pcwalton: I just don't want to make language decision issues just because of implementation convenience. 
- nmatsakis: That's not what I mean, I am thinking more of 
- graydon: I feel like coordinating efforts would be helpful
- graydon: Two places where we do each_path when reading metadata...
- graydon: Actually the most expensive part is decompressing
- nmatsakis: Can we just skip the compression?
- general answer: no
- graydon: I'm wondering if we can get away with the thinking of "this one change" and try to be more systematic
- pcwalton: I'm not saying we shouldn't remove each_path, we should.
- graydon: Can we do something systematic?
- nmatsakis: Is there more we can do than pick things to look at? I'll look at moves, Patrick and Graydon will look at resolve, and we'll check back in next week?
- graydon: I guess it's a question of time alloc. I'll write up 
- nmatsakis: it's still hard for me to understand where time goes overall, I guess I just need to spend more time digesting the data that's out there
- graydon: We were at 5 hours because for the "no-opt" tests were actually building the tests with compilers that had not themselves been optimized. As an example of the kinds of things we do, we emit an actual LLVM function call for each transmute.
- pcwalton: transmute might be fixable but doing a more general fix would require inlining in the front-end
- graydon: it seems worth doing a no-opt build and trying to find the "heavy hitters" -- what are all the things we're asking the compiler to eliminate?
- pcwalton: it's easy enough to generate compiler intrinsics 
- pcwalton: all this stuff goes down the slippery slope of optimizing in the front end, we've already started doing it, everyone does it, but it's unfortunate
- pcwalton: constant-folding and constant propagation might be valuable in the front-end
- graydon: might be necessary, if you look at what the competitors do
- pcwalton: I really don't think we should be focusing on those kinds of things yet, esp if we have outstanding feature work
- nmatsakis: I'm not sure if those are valuable, but those aren't the kinds of things we're talking about?
- pcwalton: well, we're talking about it now for if-branch removal
- pcwalton: it just seems like a never-ending rathole
- pcwalton: to some extent, inlining costs will always be with us
- nmatsakis: intrinsics we can certainly handle
- graydon: there is a middle ground as well, C++ libraries often go out of their way to limit codegen explosion
- pcwalton: I'm curious to hear some specific examples
- graydon: I'll put some notes in the e-mail I'm writing up right now
- nmatsakis: What about the option of running some tests before and some tests after?
- graydon: I think that's tantamount to turning off tests
- nmatsakis: We could make bors just stop when the post tests are in failure mode
- nmatsakis: The real question is to identify those tests that find errors only infrequently and run perhaps run those less often
- dherman: This is really an empirical question but I don't know how to answer it
- graydon: Currenly bors doesn't know how to close the tree etc
- nmatsakis: That's right, this policy would be extending bors, don't know how easy or hard this is
- graydon: In terms of hardware, though, we have seen that last year's hardware is 2x faster than this year's, so I am not sure if this is a matter of needing to do it inevitably

# Some interns are leaving

- nmatsakis: bblum, sully are leaving at least, aaron as well
- dherman: that went really quickly!
- brson: this was a good summer
- *Hooray for interns!*







