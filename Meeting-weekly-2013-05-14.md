# Agenda

* submodule ownership
* Generic paths https://mail.mozilla.org/pipermail/rust-dev/2013-April/003866.html
* fns, procs, dynamically sized types (nmatsakis)
* copy removal plans, clone (pcwalton)
* pattern @ -> ??? (pcwalton)
* Owned -> Send (pcwalton)
* removal of "pub extern" (pcwalton)
* == should auto-ref? (pcwalton)
* warn unused result (pcwalton)
* let foo = 1, bar = 2 (pcwalton)

# Attending

azita, brson, jclements, nmatsakis, graydon, tjc, jack

# Submodule ownership

- N: email sent out with list of submodules that were more fine-grained than necessary. few things were subdivided that shouldn't be. need to elaborate on what it means to be a submodule ownership. just playing tiebreaker isn't much.
- N: cases where things that shouldn't be divided were separation of traits from rest of type system. lexer / parser could be same module, but I guess it's not harmful
- G: I was trying to make the lists seem like similar size, which may not be [...]
- N: I don't know that everybody needs an equal size list. that's not a personal judgement
- J: Who were lexer and parser?
- T: Parser was graydon, grammar was patrick
- G: Assigned macros to myself because people keep coming and going who work on macros
- N: Macros were an area I wasn't clear on. Deserves an owner but there wasn't one on the core team
- G: A few areas with obvious owners but some that weren't obvious to me.
- N: wrt duties the main thing I thought make sense is a module owner should maintain user-facing docs about module. Particularly true for more outward facing modules. IDK what docs for trans.
- G: docs definitele one of them. moz page about this sugests combination of rights/responsiblitiies. making sure modules are in good state, triage, docs.
- A: other people could work on a bug in your module right?
- G: yes. it's more about responsibility. buck-stopping
- T: something about a module owner being the default person to assign bugs to.
- G: if review comes up you know who to ask for review
- A: does that sound ok to you niko?
- N: yes. I don't know precisely what modules we have, but we can hash that out
- G: [missed]
- G: if people are uncertain about submodule division, then people can reply to that email with sugestions about something that makes more sense.
- G: modules in mozilla are traditionally directories ...[missed]
- N: best to stick with code ones (perhaps?) because it's relatively concrete
- G: I'm curious to hear what's inside everyone elses head about which areas you feel are who's?
- A: Any other opinions? The original idea a couple weeks ago was to break this up so graydon doesn't need to be the tiebreaker. If you feel like you have something that you aren't cofrotable with you should bring up.
- G: If you are sitting in a meeting and we are batting around ideas in convo, who would you expect to speak up and [tiebreak]?
- G: I have to ask other people how things work all the time. It should be ovbious i'm not an expert on all these areas
- A: we are already doing this in a lot of cases anyhow
- G: merely attempt to formalize those ... maybe a bad exercise. I think mozilla is doing it because [it's a big project]. Maybe we're not at that point yet
- T: I think it would be good to formalize. I don't know where the boundaries are. "this person is repsonsible for this are". new contributors don't necessarily know who to ask
- A: niko if you feel this is too fine-grained then reply to email
- N: I did already. Those were the divisions that made sense to me.
- N: We can try and adjust, there's a feedback process.

# Generic paths

- B: bjz is not happy with what we did with static methods on type aliases. we removed something he used in his math code.
- N: What precisely?
- P: In the link above. 
- P: access static methods through typedefs. we change the grammar of paths to allow type parameters in the middle of paths, not just the end. and everything just works. you can't use them inside traits and impls (?). "everything just works".
- N: what is the problem. that it's not done yet
- B: we removed a feature he was using, so his stuff is broken
- G: we are puting in a feature that will fix this brokenness
- P: he was making a typedef for common sets of type parameters. super-generic linear algebra lib. Vec3<T>. Can't call Vec3<float>::new so typedefing Vec3<float> and calling on that. Working around the inability to have typarams in paths. Removed impls on typedefs and he was unhappy. real soln is to work around this problem and allow typarams in paths.
- N: So in your version would you be able to make typedef Vec3f and then write Vec3f::new?
- P: yes. it would "just work"
- G: can we make sure with him that this patch solves his problem. Brian?
- B: yes. I think he just wants us to keep it on our radar and get it done

# Dynamic sized types

- N: I realized that the DST proposal and ability for closures to recurse is incompatible with our notation
- N: planned to prevent them from recursing by having &fn act like &mut pointers. if you've got one it's the only one pointing to that env. in our dst proposal they would act just like & pointers and allow aliasing.
- N: thought about solutions. could write &mut fn, but it's poor notation and surprising
- N: think we should split up how closures work. have a blog post about it

- http://www.smallcultfollowing.com/babysteps/blog/2013/05/13/recurring-closures-and-dynamically-sized-types/
- http://www.smallcultfollowing.com/babysteps/blog/2013/05/13/mutable-fn-alternatives/
- http://www.smallcultfollowing.com/babysteps/blog/2013/05/14/procedures/

- P: hardwiring proc to [missed]
- B: what's proc?
- P: read blog post
- N: we have closures that capture by value and capture by ref. sort of related to ~fn &fn.
- N: @fn doesn't have that much value, ~fn useful for tasks, futures, etc. Think we should call one 'proc' (or something else) for copying. `fn` is always a borrowed closure, no sigils. dividing types up like this makes language nicer. can treat fn type however you like. doesn't act like other & pointers
- N: i get confused when I see closures that I don't know are copying or reference. maybe it's not a big problem in real code. two distinct concepts - one name.
- G: I agree there's more convo here than this meeting allows.
- P: reason I like. of all things this is the missing ingredient to having smart pointers without placement new. if you hardwire 'proc' to be ~proc
- B: So if you want @fn you just put a ~fn in an @ box?
- N: yes or use a trait. I think most places where they are written today should be traits. closure is a degenerate trait
- G: objects are degenerate fns or fns are degenerate objects?
- N: longstanding debate. good to have both. read blog post
- G: those things were seperate concepts long ago.

# Copy removal

- P: general consensus 'copy' should go away. proposal: 'clone' is a static method, not a 'dot' method. static methods do not autoderef and I think autoderef on clone is very confusing, especially since clone takes &self, so you are dealing with autoderef+autoref.
- B: why is this particular to clone not other methods?
- N: because it returns Self
- P: and because a lot of things implement it
- B: I run into this on other methods too
- J: if you have a value that implements clone and clone is also on ~
- N: I never run into this outside of traits that are "universal", like clone
- B: I have problems with newtyped structs
- P: I'd be interested to know what methods that is, since usually the method names don't overlap except for universal traits
- P: Some other things I could imagine where this might be an issue is `to_owned`, `to_managed`
- JC: It'd be good to solve this problem
- P: Since autoderef is very useful in most cases, my preferred solution is to judicuously make use of static methods where appropriate
- G: what about binary operators?
- P: these tend to fall into the category of things where the method is not overloaded as much, though in the case of == it is
- P: So I tend to think we should make that a static method as well
- P: For +, -, *, etc it doesn't seem that important to me, especially as "foo.add(bar)" reads better
- N: with the binary operators the way we treat the operator form and the way we treat the method form don't have to be the same
- N: it would be nice if the binops, when used as ops, obeyed the same rules and that rule would be 'no autoderefing always autoref'.
- Jed: operators already don't autoderef.
- N: Not exactly true, they do for the argument (rhs)
- P: they do autoderef on the righthand side. they way you should compare against a string is `"hello" == mystring`
- N: So the proposal for binary operators that I was talking about would make "hello" == twiddlestr not work. That works because the receive is &str and the rhs is auto-sliced from ~str to &str. If we adopted my proposal, it wouldn't work, because the types aren't the same, and you'd wnat to borrow one side explicitly (e.g., `"hello" == &*twiddlestr`). This way `a == b` works iff `b == a` works.
- P: My plan is: all implicitly copyable types automatically implement clone
- P: This will be implemented with some sort of magic lang item
- P: In trans when we see a call to this lang item we just reroute to trait glue
- G: Can't use deriving?
- P: I know bstrie tried it and found it was very common
- B: Isn't Eq common too?
- G: So what you're saying here is to leave more-or-less the automatic level of take-glueing we have, keep semantics of when something can be copied automatically by assignment, but map an additional method to it
- P: Yes, whatever we do we probably need what we have now, since we would never invoke clone() implicitly.
- P: Main purpose of existence is (1) to copy things with custom copy behavior (ARC etc) and (2) in generic functions, where you want to be able to copy values of some type T
- P: and we'd remove the Copy bound and just have people use Clone
- P: it's very ungeneric to use Copy bound otherwise, it'd be a function that can only be called on "implicitly copyable" things
- G: To clarify, I am quite confused, you don't want the compiler to be calling the mtehod implicitly, but with a user-defined smart pointer...?
- P: User-defined smart ponters would move unless you call clone
- G: OK, that's a nice boundary. I think that's a good thing to do.
- P: Doesn't seem too onerous right now, avoids spooky automatic copies
- G: Avoids C++ trap of compiler and library author fighting over def'n of assignment
- N:  there's two parts: keep implicitly copyable, and then there's whether  we automatically derive it. The plan for the auto if we chose it is that  there are special lang items... impls? ... such that when ... it will  choose that one when you need clone... can you re-summarize?
- P: clone() will be automatically derived for a type that is IC and non-generic
    - structs wihout type parameters in which all fields are IC etc
- P: typeck will just judge them "ok"
- P: trans will have a special case for a call to the clone() lang-item
- N: it seems like this would be another kind of method resolution
- P: I was hoping we woulnd't need it
- N: I thought that'd be easier but we can deal with this later
- B: I was concerned about static methods are unpleasant to call atm
- B: It's annoying to write Clone::clone()
- P: You can make a wrapper
- B: But you can make a wrapper to avoid the `.` notation if it's causing you problems as well
- B: I am worried about the "general trend" of wanting to move away from methods
- P: it strictly avoids autoderef hazards
- B: Maybe the problem is the hazards
- P: ok, but autoderef exists for a reason, and I don't know of a solution
- P: we could do less autoderef, but I don't think it's going away entirely
- Jack: In the case of clone, the type checker usually catches me if I do it wrong
- P: probably, true
- Jack: I feel like with `==` it's a bigger deal because you control both sides
- N: I've thought that the solution to autoderef might be the plan to make methods callable with fn form
- P: It's been on my list to bring up for a while
- G: Well we have 8 minutes now

# Universal functional call syntax

- P: this is what D calls it so I will steal their name
- P: idea is that a function taking a self parameter that belongs to a trait or an impl and a function with self as the first parameter are one and the same
- P: and you can use `dot` notation or the explicit function call syntax on either one
- P: if you use the static method-like syntax, you supply self explicitly as the first parameter
- P: though you would get the normal coercions
- P: the choice then becomes at the caller side, how explicit you want to be
- P: if you want the convenience form, you use the dot notation
- P: if you want explicit, you use the static method notation
- P: also addresses the problem where it's ambiguous which method you want
- G: I'm pretty keen on this feature particularly since it addresses the problem of having two copies of everything in vec etc
- G: if you have `x.m(y)` there are already two ways. `x.m(y)` and `f(x, y)`
- G: there's at least two cases where you want to not use method calls: want to get away from autoderef and want to disambiguate traits
- G: given those important modes and that people want to use methods a lot of the time. this is a good solution
- B: yes, this is great
- P: makes the clone point moot because it's the same either way
- G: what's holding us up on this?
- P: limited time. it's not backwards incompatible so low priority.
- G: is there anyone you can offload this work on? on IRC? all of us have our hands full
- P: I can solicit. maybe alexchricton? he's been making broad changes
- G: because it touches lib code I think someone might want to take an interest in it.

# Multiple lets

- G: any champions of this feature?
- P: niko?
- G: he doesn't like that `mut` distributes.
- P: why even make this problem for ourselves.
