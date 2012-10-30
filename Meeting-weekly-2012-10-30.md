## Attending

pcwalton, brson, dherman, nmatsakis, tjc, graydon

## Type/module namespace merge complications

  * **Patrick**: issue: now we can have types in paths, in places where they didn't used to appear before. example:

```
hashmap::HashMap<~str,int>::new()
```

  * **Patrick**: question is whether we want this, and what are the semantics.
  * **Niko**: yes! why can't we?
  * **Patrick**: this comes up because you can do this:

```
impl<K,V> HashMap { (static) fn new() { ... } }
```

  * **Brian**: ran into this yesterday when burg tried to typedef a type that had static functions inside it, and the methods did not carry along with it
  * **Niko**: not sure it *has* to work (the typedef is not the type); but seems like if you're gonna allow people to access things from anonymous trait, you're gonna end up with it anyway
  * **Patrick**: propose to solve via: make typedefs imports for their module contents, if you're importing a type path; because at that point there's basically no difference between use (possibly public) and type alias (possibly public) -- no difference from an import at that point. question is if we *want* to support that. why didn't you just write an import
  * **Niko**: it is suboptimal if it doesn't work. wonder if we need type keyword at all
  * **Patrick**: yes you do, e.g. for parameterization, tuples, @, ~, ...
  * **Patrick**: types have a structural component and need the typedef for that
  * **Niko**: yeah, that's fine. interesting
  * **Patrick**: yeah so this is a curious overlap between `type foo = ...path...` and `use`
  * **Niko**: to some extend was always true
  * **Patrick**: yeah
  * **Niko**: guess that means: if you have two impls w/ diff type parameters, can have sort of separate static functions in there
  * **Brian**: if they're the same, right?
  * **Niko**: not necessarily
  * **Patrick**: are we even going to allow that? depends on functional dependencies policy?
  * **Niko**: don't think so. example:

```
struct MyVec<T> { ... }
impl MyVec<int> { ... }
impl MyVec<uint> { ... }
```

  * **Niko**: is something like this allowed?
  * **Patrick**: would assume no, because of fundep
  * **Niko**: not related to fundep! these are different self-types for some anonymous trait, if you wanna think of it that way. always allowed overloading on self-type. that's the whole point
  * **Patrick**: guess it dpeneds where anonymous trait is. if associated w/ a nominal struct...
  * **Niko**: oh, I see what you're saying...
  * **Patrick**: this is way into the weeds; will work through it offline with Niko
  * **Niko**: I think there's more to work out with overloading; varies in different languages; don't wanna talk about it here

## REPL merging

  * **Brian**: z0w0 posted a repl using the jit; seems well-factored (in its own rusti crate, which I'm happy about)
  * **Brian**: any opinions? seems uncontroversial
  * **Patrick**: as long as it's well-factored, in a separate crate
  * **Patrick**: do need to put into CI so we don't break it
  * **Brian**: haven't looked whether he has tests set up, but we can test jit independently of repl
  * **Patrick**: at least make sure it compiles, good start
  * **Brian**: dbp is librarifying things, that might fight with rusti, both may need to be merged together

## incoming/try policy

  * **Graydon**: guess I will write here as machine is still starting up
  * **Graydon**: just wanted to get some conversation going on incoming being constantly broken. it makes a lot of other tasks difficult to have history full of broken states. bisecting is hard. doing anything with build automation is hard because most builds don't work, etc.
  * **Graydon**: wondering if we were to have sufficient quantities of machines backing try and doing incremental try check-lite builds, if we could have policy of "nothing lands w/o at least a successful try"? Is this just dependent on "enough hardware"? If so, that is something I can make happen.
  * **Patrick**: I'm ok with that generally. in general we shouldn't be scared to back out incoming changes that are broken -- it doesn't hurt my feelings, really :)
  * **Graydon**: yeah, but backing out still leaves broken changes in history
  * **Patrick**: yes, it's suboptimal -- successful fast try builds would be ideal, as you said
  * **Graydon**: ok. we just got VPN set up for amazon machines so I will try to bring Vast Quantities Of Hardware into this problem and see if that helps :)
  * **Patrick**: ok.
  * **Dave**: I love the smell of brute force in the morning
  * **Dave**: sounds like consensus
  * **Brian**: I always like more automation
  * **Patrick**: I do think in general we'll not be able to prevent *some* broken builds, it happens
  * **Brian**: if we have a policy where you have to pass try, it shouldn't happen
  * **Patrick**: in theory, but ... you get random oranges
  * **Niko**: it'll happen, but less
  * **Patrick**: happens on Firefox
  * **Graydon**: it also seemed to me that the incoming branch was set up largely to unblock interns over the summer when there was a huge landing rate, and if we're doing review-based landing anyways now, that process is already stalled / turned-down in volume
  * **Patrick**: yes, we could land on master with review and try...
  * **Brian**: what I'm worried about with try: last time we had a try branch, even though you can trash the entire branch, it's still serialized. one person uses the branch at a time. other people have to deal with them taking over that branch. you're gonna push -f every time you push to try
  * **Patrick**: can we do separate try branches for separate devs?
  * **Brian**: possibly. maybe buildbot has a solution Graydon's aware of?
  * **Graydon**: couple things to note there. first, buildbot should hold the source stamp of the try rev it's running in mind even if try branch has moved. not certain about granularity of that but it's intended anyway (modulo git gc'ing the branch, ugh). second, you can filter the console view per person, which means you can bookmark a page on buildbot that shows just your try pushes, makes it pretty easy to monitor progress.
  * **Niko**: why not try-<username>?  or pulling from our own repos?
  * **Brian**: seems like there's gotta be a workaround
  * **Graydon**: could do a bunch of try branches. gonna have to do some experimentation for queues, branches, schedulers, etc. picking a somewhat random assignment right now. expect several weeks of churn for that to settle down properly.
  * **Graydon**: only thing I wanted to discuss really: how people feel about experimenting to pursue policy that avoids a history with bunches of broken points in it
  * **Niko**: definitely in favor
  * **Patrick**: seconded
  * **Niko**: Graydon: at some point you mentioned idea of a bot that would pull (rebase), test, and push?
  * **Graydon**: that's golden state I'd like to get to eventually. more machinery in buildbot to get close to that state. can have a builder that runs on master that listens to several others, run triggered builds, can do a final step that pushes. some machinery possible in new setup. much more plausible than the one we had before.
  * **Niko**: that'd be awesome
  * **Graydon**: definitely what I'd like. everyone can post to separate try branches and a bot merges serially
  * **Graydon**: I'm grinding through buildbot process

## Versioning, re-exporting

  * **Graydon**: assuming pcwalton and niko will discuss this stuff. one note: what gets exported in a crate and what gets exported in different versions of crate...
  * **Graydon**: imagine re-exports from one sub-module for one version, then re-exports from another for another version?
  * **Graydon**: every name has a version associated with it?
  * **Graydon**: some conversation to have there, just wanted to put that out there

## Datasort refinements

  * **Niko**: still think it's a good idea. not for 0.5, but wonder if people had read my post. if not could you read it so we can have a discussion later?
  * **Niko**: http://smallcultfollowing.com/babysteps/blog/2012/08/24/datasort-refinements/
  * **Tim**: read the post, seems like a good idea, agreed it's not for 0.5
  * **Patrick**: we have plenty of stuff that needs to get done for 0.5
  * **Graydon**: I read it before, I think it's a cool proposal. not sure if square brackets are ideal, but at type level it sounds good
  * **Niko**: no opinion about the syntax
