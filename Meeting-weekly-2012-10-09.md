## Attending

tjc, brson, dherman, nmatsakis, pcwalton, graydon

## 0.4 status

  * **Graydon**: we're basically feature-complete for 0.4?

*general agreement*

  * **Patrick**: what about Windows?
  * **Graydon**: we're gonna have to fix Windows issues before releasing, naturally
  * **Brian**: intermittent failure, based on age of snapshot, probably? problem with stages
  * **Graydon**: stage 2 and stage 3 not converging?
  * **Brian**: no, stage 1 and stage 2
  * **Niko**: don't understand
  * **Brian**: Windows hacky peculiarity: on Win no rpath; set path to point to specific directory; but two dirs contain libs; one compiler, one targets; we set path on windows to one or other; libraries don't converge, not identical (symbols are wrong on one of them)
  * **Graydon**: yay bootstrapping!
  * **Niko**: surprises me; they only need same hash on metadata
  * **Brian**: yeah, and symbols need to have same names; still not clear on the required names
  * **Niko**: yeah, me neither. but at least I understand the problem now
  * **Graydon**: solving problem with giant hammer: switch snapshot to stage 3, does stage 3 still build?
  * **Brian**: not sure right now. when we do conditional compilation lately, haven't been putting in stage 3 tags at all. makes me believe we haven't built stage 3 in a long time
  * **Patrick**: let me rain on parade: windows unoptimized builds have been broken for a very long time (assert in LLVM levels of broken) -- we just don't build with opt off on bots, and I think we need to start doing it
  * **Graydon**: not so fun. what kind of assert?
  * **Patrick**: asm printer -- nasty low-level codegen assert. which might actually be straightforward to fix, dunno
  * **Niko**: usually those are pretty easy
  * **Brian**: suspect it's been like this all year
  * **Graydon**: tried reproducing on 32-bit non-windows?
  * **Patrick**: no
  * **Graydon**: I'll start a build on that
  * **Patrick**: in general we need larger platform coverage on our bots
  * **Graydon**: some time in next couple days, transition to state where anyone landing on non-release branches, if bots get backed up we'll exit them and dump your build, b/c we don't have prioritization on the builds. we will eventually

## Docs

  * **Graydon**: I did some doc work on Friday; left my edited printout at home but I'll keep working on it soon
  * **Graydon**: anyone mind suspending their activity and going into doc sprint for a day or two?
  * **Tim**: sounds good
  * **Patrick**: sure, I'm okay with taking over those duties. normally brson does that, but his work on servo is important. I'm happy to take over
  * **Tim**: enough to do on docs, a couple people could. are there particular priorities for docs?
  * **Patrick**: are there types of items that cannot yet be documented? for example, classes couldn't be documented for a long time. is that still true?
  * **Brian**: don't think so
  * **Patrick**: do impls still have weird __ extension thing?
  * **Brian**: oh, you mean in rustdoc. everything in rustdoc except fields can be documented. all items can be
  * **Patrick**: do you mostly mean rustdoc, or tutorial?
  * **Graydon**: tutorial and manual, make sure everything reflects how things work. libraries are undocumented but not *wrong*. manual is often deeply wrong.
  * **Brian**: would be good for someone to read through the tutorial. sub-tutorials are still awful except borrowed pointers tutorial
  * **Niko**: they're also pretty important
  * **Dave**: tutorial needs to be split up?
  * **Patrick**: already has been
  * **Dave**: ah
  * **Brian**: but still pretty big
  * **Graydon**: if you find yourself reading tutorial, and thinking this is more than a light treatment, then consider lifting that text out and placing it in the manual instead
  * **Niko**: a lot of people come to the channel based on what they saw in the manual
  * **Graydon**: one thing I've found useful for doc work: do a printout and sit with printout and a red pen. fantastic way of making sure it doesn't get missed.
  * **Tim**: also I find reading aloud helps me notice things with writing I wouldn't notice otherwise
  * **Graydon**: so if people can make some time for doc time, allocate a block of time *not* in front of a computer and do doc review
  * **Graydon**: do we want to divide things up?
  * **Dave**: coordinate by IRC?
  * **Graydon**: ok, sure

## 0.5 priority meeting

  * **Graydon**: niko created a wiki page with priorities list. think most of us have written something by now?
  * **Graydon**: hopefully this time next 0.4 will be live and we'll have a meeting to discuss scope for 0.5, which I'm hoping will be closer to a 2-month cycle since we won't have destabilization of summer
  * **Graydon**: so just a reminder to read the wiki and update it:
https://github.com/mozilla/rust/wiki/Note-0.5-priorities
  * **Graydon**: be prepared to talk about what you want to get done during the cycle by next week

## Review

  * **Niko**: I wanted to discuss possibility of moving to a review system, maybe for 0.5
  * **Niko**: have two pairs of eyes on every patch
  * **Niko**: don't know best specific process; but getting close to time, a lot of complex areas of the compiler
  * **Tim**: the way it's worked at other places I've been, go to someone's desk and get a code review
  * **Dave**: I like the record and formal process that we have in Firefox
  * **Graydon**: agree, especially since we have remote contributors
  * **Graydon**: main concern is of course speed; and this would slow things down some
  * **Niko**: I agree, not gonna argue
  * **Tim**: would text comments be for every single patch?
  * **Patrick**: I feel like there should initially be some threshold; small bugfixes ok
  * **Niko**: my main concern is big chunks of compiler only one person understands
  * **Patrick**: I agree; I don't think it'll be a problem with this team
  * **Graydon**: I don't think it'll be a huge problem to figure out what threshold amount will be; more concerned about potential mechanisms since GH doesn't have code review mechanism
  * **Niko**: GH commenting on LOC is nice
  * **Graydon**: it's great! it just doesn't have review mechanism -- queue and queue-per-person are specifically what I'm curious about
  * **Graydon**: separate incoming code into separate queues per person? can we create a tag per reviewer?
  * **Graydon**: so every pull request has to have a reviewer, reviewer has to comment and land it? and we start doing more of our work by pull requests?
  * **Graydon**: you can land on incoming, which means "I don't think this needs review"; social process for determining where the threshold is
  * **Niko**: seems similar to the idea of what needs to be brought up at the meeting, which we do fine with
  * **Tim**: someone should write this up in development policy page on wiki
  * **Graydon**: yeah, we'll make sure we transfer that
  * **Graydon**: at extreme end we can set up more bureaucratic policy, but I'm enjoying the low levels of bureaucracy
  * **Brian**: I have a question about reviews & policy. I'm personally not very critical in many pull requests. especially if it's far away from core of compiler. are we concerned about that?
  * **Graydon**: I'm not. I'm fine with rough sketches for library code. great thing about rough sketches is it stakes out territory. some people won't contribute until there's something there for them to fix, others prefer rough sketches. so rough sketches are helpful for the former kind of people.
  * **Niko**: makes sense for library
  * **Dave**: this is a totally natural thing; all Mozilla projects have a spectrum of core complex pieces and outer more accessible pieces

## Anonymous traits

  * **Patrick**: first of all, terminology: call `impl X` without a trait an "anonymous trait"
  * **Patrick**: question is about ability to have static methods in impls that don't have a trait -- allows `new`
  * **Patrick**: if we adopt this convention, which I like because you can say for some type T, `T::new` -- this breaks a lot of conventions, so we should think about it now
  * **Tim**: `new` as in constructor, not as in allocation?
  * **Patrick**: just a naming convention for constructor. `T::new` has precedent in Perl, maybe Ruby?
  * **Patrick**: other nice thing: other than avoiding overloading name of type, seemlessly scales to multiple constructors.
  * **Niko**: thumbs up
  * **Patrick**: also kind of nice b/c it's only one item, so if you rename the type then the constructors seemlessly rename
  * **Dave**: independent alpha-renaming :)
  * **Patrick**: yes
  * **Patrick**: we support static methods in impls, why not in anonymous
  * **Graydon**: oh, we don't currently allow this?
  * **Patrick**: right, we don't, because of resolve -- two different namespaces so no way to qualify those names
  * **Niko**: what we actually do for static functions right now: they belong to the module in which the trait was declared
  * **Patrick**: which is kind of weird
  * **Niko**: yeah. Graydon: a while back I sent an email unifying types and modules; I think this is what Patrick is referring to
  * **Patrick**: yeah, I kind of jumped multiple steps ahead
  * **Graydon**: so you're sort of treating the impl as a kind of module for the type
  * **Patrick**: the static namespace, anyway
  * **Niko**: I'd like to take this even further, but we can start with just static functions
  * **Brian**: multiple questions
  * **Brian**: right now you can define the impl anywhere in the crate, right? would seem less confusing if that was restricted to module that defined the type
  * **Patrick**: interesting. originally rules were: only define impls for a type in same crate as trait or same module as type. didn't matter whether implementing anonymous trait or not. that was considered annoying, e.g. some cases in core; can't remember off top of my head. a little more complicated set of rules than strictly necessary
  * **Patrick**: another possibility: keep same rules as now, except anonymous traits and only anonymous traits have to be in same module as a type
  * **Dave**: anonymous traits feel like a design pattern for classes. maybe something worth considering revisiting class syntax?
  * **Niko**: but this very proposal cuts down on moving parts for defining a class
  * **Brian**: well, module namespace too
  * **Niko**: okay, so a class is a struct + impl + file
  * **Niko**: I'm okay with it
  * **Niko**: I *do* think of it the way Dave describes
  * **Patrick**: precedent in Objective-C, separate code from data
  * **Graydon**: yeah, I think the separation is quite reasonable
  * **Brian**: another question: really ratchet down restrictions on self type; &Foo and @Foo surely can't take statics
  * **Niko**: those aren't anonymous
  * **Brian**: they aren't?
  * **Patrick**: that's what explicit self is for. that rule may not be well-enforced ATM. for vec it is, b/c vec can't have an anonymous trait anyway
  * **Brian**: if we have explicit self, what's purpose of impls for different variations of self?
  * **Patrick**: dont' think there's any purpose
  * **Niko**: sometimes you need to, but the way we have it in vec is not the way to do it, I think
  * **Niko**: vec is also weird b/c we don't treat vector type uniformly
  * **Patrick**: mostly have consensus?
  * **Niko**: Graydon, what's your opinion?
  * **Graydon**: seems harmless and quite reasonable to me
  * **Graydon**: don't really object to niko's original proposal to merge the namespaces, I was just afraid it might break stuff.
  * **Graydon**: just worried about pulling that lever in middle of release cycle. if it works, I don't care
  * **Graydon**: originally had just one namespace in the language!
