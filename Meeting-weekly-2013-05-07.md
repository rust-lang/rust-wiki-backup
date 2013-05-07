# Agenda
- By-value self in destructors #4330 (nmatsakis)
- Dynamically sized types proposal (nmatsakis)
- Converting binary operator traits to use an associated fn vs method #6299 (nmatsakis)
- submodule owners? (azita)
- pattern @ -> what? (pcwalton)
- copy removal plans (pcwalton)

# Attending
brson, nmatsakis, pcwalton, azita, jclements, jack, graydon

# By value self in dtors
- N: Wanted to discuss what kind of function a dtor should be
- N: Right now dtor takes &self, and I think that's not right
- N: Reason is you can't move-out of the value you're destructing
- N: I think it should take self by value
- N: this should work like any other by-value self, with the caveat that it does not invoke the dtor-for-foo in the dtor itself
- P: what happens if you move it elsewhere? won't it run dtor?
- N: yeah, that's your fault! you coded an infinite loop
- P: uncomfortable with that
- N: ok .. what are alternatives?
- P: keep doing what we're doing today
- N: ok, then you can't move from self. is that cost worth it?
- P: agree it's a real pain, but this doesn't quite feel like a solution
- N: ok. I see no problem running a dtor twice, happens in other languages
- P: so far we've tried to avoid it! why do we even have the drop flag?
- N: so "we" == compiler don't run it twice
- (back and forth disagreement about morality of running dtors twice)
- G: related to recursive failure
- G: the way I wanted to solve this was to have a recovery mode where we didn't run user code. it's a dynamic process where if you try to drop it again we just do the most neutral thing possible
- N: since dtors are syntactically special, we have special one-off rules, we could have rules that say you can't move self as a whole.
- B:  does this destructuring-self issue come up often enough that it'd be to  be obnoxious to have a mutable self and tear off option types during  shutdown, just to take apart?
- G: currently we track if an object has been destroyed in a flag in the object?
- N: yes. it's a hidden field
- G: if that were in a 'try' state, being destroyed. would that solve a problem here?
- N: we're hoping to to get rid of it. the compiler should have enough knowledge to know when to run the dtor
- G: maybe conflating recursive failure / recursive destruction ...
- G: failing a second time would reenter the stack without running dtors. we run dtors on two exec paths, one during normal execution one during unwinding
- N: we're in a drop trait we have a special rule on self. when you move something it can't be self
- G: but it can be destructured
- G: we don't normally run a dtor when moving an object
- N: you want to enforce a rule that values are actually 'getting smaller'
- N: the moves pass already computes whether moves are complete or partial
- J: what if you just moved all the pieces out of self and constructed a new self. seems to be the same thing?
- N: I agree
- G: that's less likely to do by accident
- J: if it becomes a common idiom to reassembly self in a dtor ...
- N: I don't expect that to be the case
- N: What i expect to be common is to send something on dtor
- G: Do you think that static check can be done?
- N: yes

# Dynamically sized types

- N: Does anybody have any thoughts?
- G: Scope and timing worries me. Sounds like a good idea
- N: If we're going to do it we should. it affects everything
- P: It's so good we can't not do it. Fixes a wart
- N: It's a frequent confusion.
- N: It's unclear how often we will write the Sized bound, and there may be complications
- G: I'm concerned about the amount of time it's likely to suck up. Is there any way to estimate?
- N: I have no way to estimate
- G: Take a week to sketch?
- N: I guess you would run into any problems fairly quickly. Touches various parts of compiler
- N: You could imagine we keep current types and if you use parens explicitly it switches to newer types
- G: I think you'll probably just want a big 'staging' switch
- N: We have special rules about ~fn and what it can close over. Might need to add `~fn:Owned` to be explicit.
- B: Same for objects
- G: You have to write a bound afterward. If you don't it captures by ref?
- N: No, it closes over anything but isn't sendable.
- N: The way we distinguish between copying closures and by-ref closures is buggy but it should work ok. You can't have ... <missed>
- G: Do you have another major problem this would interfere with?
- N: Mostly cleanup and other fixes. Main thing is non-rust work. Maybe Patrick.
- P: I have a pretty long list of language features but can prioritise this.
- <boring stuff>

# Binops, fn vs method

- N: Right now we use a method for binops which makes them asymetric. If we used a binary function it would work more smoothly.
- G: Do we have associated functions working?
- N: They work in the most common case, when `Self` appears in 
- B: We don't rely on special method dispatch rules for binops?
- N: In fact we kind of turn them off
- G: Any risks?
- N: Noby writes `x.add(y)`?
- B: We have a lot of traits that are 'alternate' binops, Equiv, TotalOrd? Would they be methods?
- P: Probably not
- N: Don't know
- B: If you were going to write long form you would want to use methods. Could we leave the methods and add functions for the lang items?
- N: For macros and things you would want to not use methods to avoid gotchas with auto-* behavior
- P: x.add(y) is nicer than add(x, y). Puts the operator where it should be.
- N: We could leave them as they are and add a function like this for macro authors or whomever:
    fn eq<T:Eq>(a: &T, b: &T) -> bool { a.eq(b) }
- N: I think I withdraw this idea.

# Submodule Owners

- G: I've been sticking my fingres into conversations where I don't feel equally versed
- G: I've been feeling like it would be more reasonable and healthier for project
- G: If we formalized some sense of submodules and had people who are not me having final say on things
- G: Really formalizing in a written down sense existing roles people are already playing in project
- Azita: Sometimes if it is hard to come to a conclusion, there is a need for a tie-breaker role
- Azita: Makes sense to have owner of relevant module make that decision
- Azita: help to share work as well
- G: It's somewhat artificial as well
- G: Borrowing terminology from mozilla project, in terms of modules and owners, code reviews, etc
- G: But I'm curious if anyone feels like this is a bad move
- B: So there are two goals: one is to ease Graydon's burden and the other is to break ties
- N: Maybe also a goal of keeping design in your head
- JC: I think code ownership is a good idea
- G: Projects differ, some projects feel everyone should feel collective ownership
- G: But I don't think we're talking about saying that one person has to fix all the bugs in a given area but more final say in a technical conversation where there are numerous viable options and a judgement call must be made
- G: We usually make decisions as a team, and I think we will continue to do so
- G: But sometimes we have to end conversation with something we're going to try now
- G: But often I feel like I wind up in a position of ending conversations and I don't feel like that's necessary for me to be doing it
- B: So I think the practical effect of this is that we should think twice before deferring to Graydon to break ties
- Azita: Right, nothing would change in terms of overall management
- B: So I feel like a lot of times when I defer to someone else, it's because I feel like they are the module owners, and should be the ones to break the tie
- B: I think this is a good idea but it's not clear to me if it is solving a problem that we have or not
- B: I haven't noticed these types of stalemates occuring that frequently
- G: It's more that I feel like I'm artificially inserting myself into decisions
- G: ok, i'll send an e-mail with my first thoughts on the topic
