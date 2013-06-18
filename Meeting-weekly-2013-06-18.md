## Agenda

- Tree maintenance, new bors/auto builders
- Prep for 0.7, milestone refresh
- Switching to http://benjamin.smedbergs.us/weekly-updates.fcgi/ ?
- once fns
- language def'n: namespaces.  see e.g. https://github.com/mozilla/rust/issues/5792 , also 6894

## Attending

graydon, felix, sully, jack, niko, eston, bblum, aaron, ecr, azita, brson, tjc, jld, tikue

## Tree maintenance

- graydon: tree closed to get better test coverage. too much breakage. turned on a bunch of new builders. not real machines but just different configurations. optimized, unoptimized, cross, valgrind builds. should put out fires but slow down the landing cycle.

## 0.7

- graydon: milestone says we're complete at end of month. everyone go back to 0.7 milestone and clarify things you are going to get done in the next ~week. if you're not going to get it done then remove it, if it's important than make a note of it. it's ok - don't feel bad
- graydon: release notes are started. if you have time go through them and make sure they reflect what you did. detailed release notes are empty.

detailed release notes: https://github.com/mozilla/rust/wiki/Doc-detailed-release-notes  
0.7 doc issue: https://github.com/mozilla/rust/issues/7097  

- graydon: think this is a really good cycle. converging on a backward compatible language. didn't expand the scope this time.
- azita: graydon, are you going to have a seperate meeting for 0.7?
- graydon: there are 37 open errors and every one is a assigned. individuals should look through those and triage etc. be realistic. by this time next week that list should be close to done. it's not huge. previous releases we've had ~200 open bugs, none assigned to anyone.

## Weekly update software

- graydon: jdm asked why we were'nt using ben smedberg's weekly update thing. I just forgot that it existed. this is apparently what the rest of mozilla does for weekly status updates. this is public so different from our current system
- felix: what's the workflow for replies?
- graydon: idk. not sure if there's anything beneficial here - well, it links to bugzilla, but we're not using it. it does give more insight into what we're doing. want to try it for a couple weeks? ok, everyone make an account.
- niko: you have to ask a developer for an account?
- graydon: oh, we'll figure it out

## Once fns

- bblum: want to make a case for putting them in the language
- bblum: went through runtime, servo, etc. found places where stack closures needed Cell
- bblum: argument against was traits already express onceness so you wouldn't need special type for heap closures. for stack you would
- pcwalton: why?
- bblum: for stack closures that need to move out
- pcwalton: &trait?
- niko: can't consume &trait, but you can pass by argument
- bblum: here's some stats about most frequent stack closures that need oncification

https://github.com/mozilla/rust/issues/2549#issuecomment-19588158

- bblum: all fns involved are used in both once and non-once styles. so if you converted it to take an arg and thread it through closure then you would have callers using (). forcing those that don't need onceness to write extra parens or duplicating library to provide both signatures. think disadvantage outweighs that of having extra keyword. argument against was that 'once' kw would lead to confusion, but I feel that having library duplication or extra () would be moreso.
- bblum: have a local branch that implements once closures
- brson: does this include once unique closures?
- bblum: if unique closures are removed and replaced with traits, then they can be handled with a fn(self) declaration
- brson: so this does exclude heap closures?
- bblum: this is for stack closures

```

Proposed:

fn map(x: Option<T>, blk: once fn(T) -> U) -> Option<U>

let some_opt = ...;

let result = do some_opt.map |val| {
    channel.send(arc_handle);
    port.recv(result)
    return result;
};

What you have to do today:

let arc_cell = Cell::new(arc_handle);
let result = do some_opt.map |val| {
    channel.send(arc_cell.take());
    port.recv(result)
    return result;
};

The passing argument pattern:
let result = do some_opt.map_move(arc_handle) |arc_handle, val| {
    channel.send(arc_handle);
    port.recv(result)
    return result;
};
let result = do some_opt.map_move(()) |(), val| {
    return result;
};
```

- graydon: any place right now where you wanted a once fn you'd end up duplicating the api?
- bblum: right
- niko: there are a bunch of places where onceness is used, like in the sched, where only experts are going to be touching
- bblum: i see the scheduler as a test for low-level programming, others will run into it
- pcwalton: every language with affine types and closures also has one-shot closures. bucking trend has been painful. other languagess *don't* have moving into closures
- niko: that's what once fns are
- pcwalton: was thinking more of the diff between &fn and ~fn
- niko: i personally don't mind parens. if you're writing something like a scheduler you need to know rust pretty well so can tolerate extra parens, you know their purpose. on the other hand, once fns do address this case nicely.

- bblum: if you are calling them on a trait you don't get the guarantee
- pcwalton: that's unfortunate
- nmatsakis: probably you want two traits, one for arbitrary length sequences, and one for maximal 1 length sequences, and the latter extends the former
- pcwalton: that sounds more principled

trait Mappable<T> {
    fn map<U>(self, blk:      fn(T) -> U) -> Self<U>;
}

```
impl Mappable<T> for Option<T> {
    fn map<U>(self, blk: once fn(T) -> U) -> Self<U>;
}
                      // ^ stronger guarantee

Or perhaps:
trait OnceMappable<T> : Mappable<T> {
    fn map<U>(self, blk: once fn(T) -> U) -> Self<U>;
}
```

- graydon: I want to take a slightly different angle, we currently wind up duplciating APIs for by-value and by-reference. One of which expresses contexts where you can move something in or out of the container and one where you can't, and they mean somewhat different things. I don't view it as a cognitive burden to the user since they are already confronted with this, where there are APIs. I find the concept of moving out of the environment due to environment capture a more likely cognitive hazard than by argument passing. We already kind of do that, we already have APIs where by providing the argument you are giving it up (e.g., hashtable insert).
- nmatsakis: if we move to external iterators, then the use case of each/map is a non-issue.
- bblum: not for map, you definitely would take a closure there.
- nmatsakis: true. it works for each though.
- bblum: I will point out that none of the cases I pointed out are cases that would be replaced by iterators.
- jack: pattern in servo is to use these things when we spawn tasks. A lot of times we pass in channels for different pieces to communicate with each other.
- pcwalton: this is different, those are unique closures, but this is just for stack closures.
- bblum: same pattern but it would only be fixed for unique closures.
- jld: something i'm wondering and I don't know how interesting this is. we're talking about affine functions when you would like to deinitiatilize, but what about relevant functions to initialize upvars?
- bblum: there is some demand for that. For example, in the new scheduler, there is this pattern `deschedule_running_task_and_then`, frequently used to store a value in the caller's stack frame. Much less frequent but I don't know.
- nmatsakis: it's tricker, harder to guarantee that something does happen than to guarantee it doesn't happen more than once
- bblum: I thought the way this might be done is to have a a trait, well, maybe not a trait, but a library type that comes with a destructor where the destructor calls it in case you didn't
- nmatsakis: sort of unsatisfactory
- bblum: if it were a language primitive, it'd have to come with its own destructor,
- nmatsakis: I'd prefer if it were just not guaranteed in the case of failure
- nmatsakis: I do have a use case for this pattern but I think it's a separate issue we should discuss separately
- jld: I was just wondering how big a can of worms is hiding under this once keyword
- pnkfelix: I was wondering if we should just reserve the once keyword and not use it yet?
- pcwalton: seems prudent, it's not clear if we will decide this here. I will mention that the two use cases in servo I'd prefer to replace with an RAII pattern, mostly to avoid rightward drift.
- graydon: I think it'd be easier to decide this if we were to modify the APIs to use argument passing rather than cells, since that's the expected remedy. We're talking about six or either functions.
- pcwalton: do keep in mind that the scheduler is not done, so I could foresee more such functions appearing.
- ecr: it's also appearing a lot in the network I/O code that I'm working on. For working with sockets, you wind up with various stack closures and you use cells to pass results around (ed-approximated). Ugly.
- pcwalton: On the other side I can see `exclusive.with` being replaced with RAII
- bblum: No, they require 
- nmatsakis: I don't think you can do it due to the limitations on what can safely have a destructor.
- nmatsakis: Wouldn't want to allow the object that represents the lock being held to go into a managed box
- brson: So you can put a region pointer into a managed box?
- nmatsakis: You can but you won't be able to access it, unless there were a destructor, which is why we disallow that
- ...
[pcwalton questioned lifetime for regions vs. lifetime for destructors in the presence of GC around here, I think. — jld]
- bblum: these interfaces need to be implemented with an unsafe dtor
- pcwalton: I'm wondering whether this causes problems for `with_imm_text` as well?
- nmatsakis: I don't think this causes problem for the use case as you described it, which was to pair a managed box with a pointer into the box
- graydon: getting far afield. I'm not going to bring down the hammer here but I really feel like, if we can emulate it, we should. This feels like another case where we sometimes have to duplicate APIs. I'd be willing to revisit the question.
- pcwalton: maybe we should just reserve the keyword.
- nmatsakis: I think I tend to agree with graydon, though I think once fns are a kind of nice technical feature.
- bblum: I feel like the workaround will be more confusing for newcomers.
- jld: could we use default methods to make implementing the duplicate api easier? assuming they were working?
- nmatsakis: yes.
- jld: that would help address cases where people forget to write it
- nmatsakis: not really, it's always up to the author to advertise that the fn is only called once
- graydon: ok, for now let's reserve it, I'm concerned with adding new features and I want to see if we can live without it, I understand it is common

# Namespaces

- pnkfelix: I've been trying to figure out our situation with namespaces. Module and types share the same namespace. The language is written in a way that makes it sound like the only conflict you will have is between modules and types, but there is also the possibility of two types shadowing each other. I am confused about when shadowing is allowed, can types shadow one another? What about this example:

```
struct C { a: int, b: int }
enum Foo { C { a: int, b: int } }
fn main() { }
```

- pcwalton: This should  be an error. You've stumbled upon the fact that struct-like enum variants are weird. 
- brson: It's strange that we have some variants in the type namespace and some in the value namespace.
- pcwalton: We could also say that enum-like variants are in both the type and the value namespace, same as structs.
- nmatsakis: Do we need them?
- brson: I've wanted them a few times, but they don't work reliably
- pcwalton: I like that it makes the things that appear after a struct and a variant the same. Feels simpler to me.
- pnkfelix: ok, I was worried I was being stupid, but in fact this is complicated.
- pnkfelix: I am right in thinking that the struct namespace is sort of a ... sub-namespace of the type namespace?
- nmatsakis: I think the idea is that there is this 'meta' namespace, inhabited by types, modules, and struct-like variants, not just types.

- pcwalton: Let me draw an ASCII art diagram!

```
                   module def
                 /
        type NS
      /          \
 name              type def
      \
        value NS - value def
```

- jld: fwiw I used a struct-like enum variant at one point because I had too many fields with types like bool and int and it was getting confusing.  And we had this feature to give this names.
- nmatsakis: I usually just make a struct and wrap it in an enum, because that allows me to write helper functions that know that they are dealing with that specific variant.
(jld quietly realizes that that approach might have made the code more readable, as it would allow field projections instead of just pattern-matching.)
- graydon: out of time, thank you all for coming.







