# Agenda

- OSCON report
- for .. in .. { .. } plan (graydon)
- overloadable * (graydon/patrick)
- fate of *: lifetimes, library? (nmatsakis)
- precedence of "as" (pcwalton)
- module system visibility bugs (graydon)
- warn unused result (pcwalton)
- IndexAssign (pcwalton)
- == and autoderef (pcwalton)

# Attending

azita, pcwalton, nmatsakis, pnkfelix, graydon, jack, bblum, shu, brson

# OSCON report

- J: went very well
- J: fancy booth, Brian and Jack spent all day answering questions
- J: most people wanted to know what programs use it
- J: much socializing, getting to know the Samsung folks
- G: anyone go to Dave's talk?
- J: Yes. Both talks had good attendance, Dave's slightly better attended.
- J: Also went to the PDX Rust meetup. SprocketNes, phantom types. Very enjoyable.

# For .. in .. { .. }

- graydon: i implemented it with syntactic lowering and it doesn't go very far into the compiler. i'm going to do some more work on this and was wondering if anyone has any objections.
- graydon: in order to do that you have to add it to the ast and keep it in the ast forever. i started working on doing the full translation of it.
- nmatsakis: +1 to doing it in the frontend.
- graydon: i'm not sure to what extent this is a generalizable strategy. it generates a fair bit less code in the IR sense. we don't have to generate closures and eliminate them.
- nmatsakis: what is the exact code your'e generating?
- graydon: it generates a temporary and a match expression and breaks on the none case.

# overloaded dereference

- pcwalton: i wasn't going to bring it up since i sent it late last night. people might need more time to digest it.
- graydon: we'll discuss it more on the mailing list. i always thought this operator was just on the side of too far, but perhaps it's not.
- pcwalton: this could unify multiple features in a single feature. i like killing multiple birds with one stone.
- nmatsakis: i need to think about it some more and haven't had time yet.
- pcwalton: it's basically the ?? pointer trait under a different name

# fate of *

- nmatsakis: erickt had a pull request that added lifetimes to unsafe pointers. there was another email about removing unsafe pointers altogether and put them in the library. i wanted to poll the group about this. i guess the question is how safe do we want the unsafe pointers to be.
- graydon: i'm pretty much fine with how they work right now.
- brson: i'm also fine. i don't really understand the motivation for adding lifetimes.
- nmatsakis: sometimes you make a pointer and you know it's safe to use but you can't control what happens if it were to escape.
- brson: why wouldn't you use a borrowed pointer in that situation?
- nmatsakis: it might be nullable. but then you could use an Option<&> in that case.
- pcwalton: this seems like a very specialized use case. it feels like this could be a library type.
- nmatsakis: if we guarantee that optional borrowed pointers have the same representation as *T then we could do that.
- brson: it seems fine to cast to a rust pointer if you can guarantee the semantics are the same as a rust pointer.
- nmatsakis: the example that he was giving was something that worked with c strings specifically.
- pcwalton: we need erickt in this conversation
- metajack: I haven't had any problems with unsafe ptr beyond bindgen generating *t instead of *mut t
- graydon: erickt: can you possibly elucidate the motive a bit more?
- erickt: sure. I found a couple places that were using str.as_c_str() and saving the interior pointer, but because .as_c_str may alloc a temporary null terminated string, this was unsafe. it just happened that the temporary wasn't getting stomped on by other allocations
- nmatsakis: it's possible in that particular case that as_c_str() should provide an &char vs *char
- nmatsakis: i propose that we have this conversation online instead. mainly i wanted to know if anyone else thought this was a problem.

# precendence of `as`

- pcwalton: there has been a lot of talk about the precendence of as being wrong. it's in between + and * right now. there are a number of arguments about where it should be but i think it's indefensible where it is now. it's the worst possible place. what happens is that whenever someone was confused they patched the compiler to change it, so it bounced around all over the place and now it's in a ridiculous place. i prospose to make it bind very tightly.
- graydon: together than unary *?
- nmatsakis: in between binary and unary operators.
- pcwalton: the tightest binary operator
- brson: what about `in`?
- graydon: i imagine that would be at the other end
- pcwalton: what's the precende of @ right now? basically as would be the same as @ is today. unary operators bind tigether than binary operators.
- brson: having them be the same seems intuitive to me
- graydon: i have no strong feelings on thsi at all
- pcwalton: having it at the same precendence of @ seems good because that's never surprised me.
- nmatsakis: it was proposed that we should change @ ???
- pcwalton: i was thinking that as should bind less tight than `.`. 
- nmatsakis: '.' binds tighter than unary.
- pcwalton: that seems right to me then.
- graydon: any disagreements?
(none)
- graydon: i would have thought of it as an extremely loose binding, but i don't have a strong feeling at all.
- pcwalton: where does instanceof bind in java?
- graydon: instanceof is level 6, < and >.
- pcwalton: that's weird. that's not quite as bad as between + and *. it follows C where the cast binds really tightly.

# Visibility in module system

- graydon: this comes up regularly on irc. how do we do crate local visibility. i can't access sibling module contents. i thought we designed the rules to permit this.
- pcwalton: it's just a bug
- graydon: we have the ability to reexport sibling privates? i'm trying to remember what the pattern is.
- pcwalton: you can reexport child privates.
- brson: you can use a private child right?
- graydon: the pattern is the parent module makes private child. we designed it so that the pub and priv words control your ability to cross a module boundary.
- nmatsakis: idea was that the parent would pub use things from the child, but child is not itself public, right?
- graydon: yeah
- brson: the parent can also priv use to expose things to other parts of the crate?
- graydon: children should have priv access to the parent?
- (everyone says yes)
- graydon: just want to make sure we're not opening a can of worms adding crate local visibility

# warning unused result

- pcwalton: there have been several bugs that a warning on unused result (unless annotated that it was ignorable) would have caught. since we sometimes use returns for error handling, it makes me nervous to not warn about misusing them.
- pnkfelix: who gets the annotation? the type that should not be dropped or the function that you are calling (or the caller?)
- pcwalton: the annotation is on the actual function which means it will fall over for HOF. arguably for HOF you should be checking the return val anyway.
- nmatsakis: you could make it a standard lint, and then the caller would need to be annotated. or you could do something different and 
- pcwalton: i was proposing both. it's your choice whether to have it on and if it's on you can annotate the callee.
- graydon: with a builder pattern you want the api to annotate itself. the final link in the chain will always be a throwaway.
- pnkfelix: "how feasible would it be to attach the attribute to a type (rather than particular caller or callee functions)."
- nmatsakis: you have a lint that turns it on and you find the expressions and check what their type is. it's not that hard. 
- nmatsakis: the hardest part is distinguishing how pervasive the annotations are. e.g., if T should not be dropped, should Option<T> be dropped, etc?
- graydon: i think it's worth trying to experiment with. we'll see how annoying it is.

# index assign

- pcwalton: there's an index trait and it doesn't work very well for assigning new values. you can have a hashmap and assign a new value. it would be nice to have an indexassign trait so that you could say hashtable[foo] = bar. right now index always returns a reference so you can't do that. this is similiar to AddAssign and SubAssign
- nmatsakis: index is wrong with other ways. it should probably return a borrowed pointer. you have this distinction between lvalue and rvalue.
- nmatsakis: &foo[x] --> if foo is a vector, this points into the vector
- nmatsakis: &foo[x] ---> if foo is something with an overloaded [] operator, it's equivalent to:

    let tmp = foo[x];
    &tmp

- nmatsakis: not at all the same thing
- nmatsakis: probably need an &mut variation as well
- pcwalton: it definately needs to return a reference
- nmatsakis: but &mut is not enough since we don't want to do the C++ thing of returning pointer to uninitialized data
- graydon: i agree this is very simliar to add/subassign
- pcwalton: i remember strcat and i worked out a three trait solution to all these cases, but i don't remember them off the top of my head.
- graydon: in general i would tend towards more traits and less cleverness. i think c++ has few definitions here and they aren't doing themselves any favors.

# `==` and autoderef

- pcwalton: currently == only auto-derefs the left hand side. if you have a ~ pointer then this works:
    let x: ~str = ...
    "foo" == x // works -- desugars to "foo".eq(x)
    x == "foo" // doesn't -- desugars to x.eq("foo")
- nmatsakis: there is different treatment of the LHS and RHS.
- pcwalton: the reason the other one doesn't work is that eq wants the same thing on both sides. the reason the LHS works is that it desugars to (see above). that works because you have &str on LHS and it borrows with coercion to &str. the other doesn't work because you can't borrow &'static str to ~str. so this is really strange. and since comparing strings is seomthing everyone does, this leads to bad experiences. i don't know off the top of my head how to solve this.
- nmatsakis: i want to treat operators differnetly so you don't apply coercions to the arguments, but that makes neither of those examples work. but at least it's consistent.
- pcwalton: i posted to the amiling list about that, and people were screaming murder.
- nmatsakis: people on IRC seemed to like it. i dont' know if we're going to be able to make them both work.
- pcwalton: you could have an ad-hoc rule that you try to borrow one side to the other.
- graydon: do dynamitcally sized types help here?
- nmatsakis: i don't think it makes much difference. are you proposing if one side is & and one is ~ we borrow always?
- pcwalton: yes
- nmatsakis: yes. we could do that.
- pcwalton: it's pretty ad-hoc and gross but it does make this work.
- nmatsakis; we could do that and my proposal.
- pcwalton; i don't think it makes sense for other binary ops because the LHS and RHS might be different types altogether.
- graydon: what if we get rid of that constraint?
- pcwalton: i think that messes up the type sig of eq. suddently they will have to say T: Eq<T>.
- n: it's nice to write Eq impls that only need to compare against themselves and not other types
- pcwalton: we have the equiv trait. we could make == equiv but it seems weird. i'm nervous about making it non-symmetric.
- brson: what if we didn't do the borrowing but ??
- pcwalton: i'm nervous aobut that. java got in trouble doing that. i ddon't know of any trait system where eq is not symmetric.
- graydon: it comes down to borrowing. when do we auto-borrow things?
- n: there are also limitations in trait matching (no backtracking) that will influence how well Equiv<~str> and Equiv<&str> will work
- pcwalton: we have equiv and it's defined with exactly those things in mind
- graydon: how bad would it be if we turn off borrowing on == and != and say they have to be exactly the same type and we wind up telling people you want to do equiv because it does borrowing
- pcwalton: that is the conservative thing. maybe we should do it and see if people complain.
- jack: several languages have == and === and one of them does coercions.
- graydon: can you summarize this and get some community feedback?
- pcwalton: i really don't like == vs. ===.

















