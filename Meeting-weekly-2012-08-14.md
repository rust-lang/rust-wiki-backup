# Attending

Lindsey, Niko, PaulS, Eric, Patrick, Elliott, Brian, TimC, Graydon, Sully

# Labeled break and again

  * **Patrick**: I think there's consensus this is a good thing
  * **Patrick**: trying to implement all the syntax we want over the next week or two
  * **Patrick**: question is what is the syntax
  * **Patrick**: brson proposed `loop: label ...` which seems as good to me as anything else; C syntax doesn't work b/c of ambiguity with record labels
  * **Niko**: also `while: label` or just loop?
  * **Patrick**: probably while, why not; but question of whether before or after condition
  * **Graydon**: I'd like colon after the label
  * **Patrick**: no opinion
  * **Paul**: could write `break break break` to go out three levels
  * **Dave**: no, absolutely not
  * **Graydon**: that's absolutely horrible, and we want sideways jumps
  * **Graydon**: LLVM hates tail calls, so this is important for direct jumps; direct dispatch gets perfectly branch predicted => fastest way to do state machines
  * **Graydon**: could implement a generalized goto... everyone likes that, right?!
  * **Niko**: there's a critical difference in expressiveness between labeled break and continue vs goto
  * **Graydon**: what I wanna be clear on: if you could continue into a *non-enclosing* loop, that's like goto
  * **Dave**: why not just call that goto
  * **Niko**: yeah, that *is* goto
  * **Niko**: why would we wanna support that?
  * **Graydon**: for coding state machines?
  * **Niko**: I don't wanna support that :)
  * **Niko**: I would rather have tail calls
  * **Niko**: I feel like that's another feature that's separate
  * **Patrick**: do we know that LLVM won't optimize switches? we could just add that optimization to LLVM, to transform that into the goto-based version
  * **Niko**: so long as you assign the variable in some obvious way
  * **Patrick**: I feel like that's something LLVM could do, and if it doesn't, that's a pass I know how to write
  * **Patrick**: that's almost clearer code than jumping via continue
  * **Niko**: strongly agree
  * **Graydon**: so if you were to write a constant to a variable and then immediately find yourself switching on that variable, it would jump to the switch target?
  * **Patrick**: yeah
  * **Niko**: we could do that in trans
  * **Patrick**: LLVM's a better place for it
  * **Dave**: I've heard different stories about how feasible tail calls would be
  * **Patrick**: we do sibling-call optimization
  * **Eric**: do we have any guarantee?
  * **Graydon**: it's an optimization LLVM does sometimes
  * **Eric**: Scheme makes it a guarantee
  * **Graydon**: not possible with LLVM
  * **Dave**: Patrick seemed to think differently
  * **Patrick**: I'm not convinced we couldn't go to Pascal calling convention
  * **Graydon**: we've been through this conversation a bunch of times
  * **Graydon**: two problems:
   * general purpose calling convention cost
   * leaving a frame is an effectful event in Rust; you have to syntactically disambiguate whether you want those effects to occur before or after (that's why we had the be keyword in the first place)
  * **Graydon**: we went with syntactic disambiguation, then banished b/c nothing was using it
  * **Niko**: that makes a lot of sense
  * **Graydon**: leaving a frame is effectful; it runs destructors
  * **Tim**: ISTR one reason no one was using was b/c of how passing-by-alias used to work; I don't know if borrow-checking will handle that better
  * **Graydon**: borrow checking will have to know; it changes a lot to go in that direction
  * **Niko**: I definitely agree it would be a new keyword
  * **Dave**: important benefit of tail calls is modularizing iterative algorithms, and can be made pay-as-you-go; but I don't know if LLVM makes that impractical to implement
  * **Patrick**: once you have destructors, nothing is a tail call
  * **Niko**: that's the point of `be` -- it tells you "can't do it"
  * **Graydon**: I agree it does allow you to modularize state machines; the limit of that is that it doesn't allow us to modularize state machines across dynamic objects: the way the PLT works it just doesn't work
  * **Graydon**: we're really only talking about ability to do calls inside of a single crate
  * **Eric**: what about with CCI
  * **Graydon**: if it's been inlined that's the same as being in the same crate
  * **Graydon**: so you're talking about modularizing from function level to crate level, which is significant, but...
  * **Niko**: you could also annotate functions with attribute that requires they be tail-callable, but yeah, it's a lot of machinery
  * **Brian**: we don't have to change the calling convention everywhere, do we?
  * **Graydon**: either that or compile every function twice; or compile every tail-callable function twice
  * **Graydon**: I'm not saying it's a bad feature, just saying that the benefit has to be weighed against the cost
  * **Patrick**: my honest feeling is that this is a Rust 2.0 feature; don't want to stop discussion on it
  * **Patrick**: we all agree we need labeled break and again
  * **Dave**: I suggest:
   * no `be` for now, but reserve the keyword
   * add labeled `break` / `again` but not with sideways jump
   * investigate LLVM optimization
  * **Graydon**: can't put labels anywhere though, syntactically
  * **Niko**: I agree with Dave we shouldn't have a fake goto; I don't want goto b/c everything we do in the compiler has to be converted to a control-flow-graph check; a lot of our block-based reasoning won't work anymore
  * **Graydon**: can you elaborate?
  * **Graydon**: you're talking about the safe version of goto that Java implemented: what's that?
  * **Niko**: breaks out of any enclosing block, including loops; maybe similar to what you wanted; main thing is you can't jump into the middle of a loop
  * **Graydon**: agree that I don't want that
  * **Patrick**: also can't allow jumping into the middle of match
  * **Paul**: or jumping past let?
  * **Eric**: should be ok
  * **Niko**: liveness could conceivably handle it
  * **Sully**: C++ doesn't allow that
  * **Niko**: so, not inconceivable, but definitely generalizes control-flow in a significant way that will affect borrow-check and liveness
  * **Patrick**: if we're going to implement today, I don't think sideways jumps are an option
  * **Patrick**: but we need to not close that door permanently via syntax
  * **Niko**: hokey in Java that you can label any statement but if it's a loop it has funny semantics; could say just add labels to loop for now
  * **Patrick**: with brian's proposal there's syntactic space for labels on loops but not in other spaces
  * **Niko**: use syntax
  * **Patrick**: but that would be different
  * **Niko**: that's a feature: you don't break to them, you goto them
  * **Niko**: I think Graydon proposed you could again to a loop that's not enclosing; that would break out of any number of loops and jump ahead to some sibling
  * **Patrick**: I feel like goto is clearer in that case; you haven't done it yet, it's not again!
  * **Graydon**: which makes me think the keyword should just be `loop`
  * **Patrick**: I'm okay with that; significant number of people don't like `again`, shaves off a keyword, makes this possible to do without closing the door
  * **Niko**: does that mean the code is gonna be, you'll have a loop that you run just once, to simulate goto? that sucks
  * **Graydon**: seriously, how many times do people use goto?
  * **Niko**: okay, yeah, sure
  * **Graydon**: allows you to recycle syntax and generate the idiom, without having to support the goto syntax and restrict it
  * **Niko**: if all we want is to avoid switch overhead, I'd say just optimize that
  * **Dave**: seems like the most conservative approach
  * **Graydon**: fine with that, but let's leave the syntactic space to go with sibling jumps if the optimization doesn't work
  * **Patrick**: with brian's syntax, we're closing off ability to goto arbitrary statements, but not closing off ability to do semantically same thing by sibling loop jumps; are we okay with ...
  * **Dave**: no, as N says, we could have a new kind of label
  * **Niko**: or we change this corner of the syntax, which is hopefully not widely used
  * **Patrick**: here's the proposal:
   * adopt brian's syntax: `loop: label` , `while expr: label` ...
  * **Brian**: do it just for loops; can embed a while in the loop
  * **Patrick**: ok:
   * brian's syntax: `loop: label`
   * in order to allow sideways jumps in the future
   * allow break and again to specify a label, but has to go to enclosing loop
   * investigate LLVM optimization of switch
  * **Niko**: someone on IRC says it does not exist already
  * **Patrick**: ok, but I really think it'd be easy to add
  * **Graydon**: I'd prefer loop label:

(general agreement)

  * **Niko**: one more token of lookahead
  * **Patrick**: no, it's not
  * **Brian**: don't even need the colon
  * **Dave**: it's nice visually
  * **Eric**: it might be nice without
  * **Dave**: helps distinguish different keywords
  * **Patrick**: I actually think the colon is good, distinguishes from some boolean expression
  * **Graydon**: it's just like the C and assembly syntax

# Common fields in enums

  * **Patrick**: I'm not super happy with the syntax
  * **Patrick**: have an idea that may or may not work, but doesn't involve extra syntax
  * **Patrick**: currently if x is an enum, can't write x.foo; what if we said that if all of the variants of your enum are record-like and have a field called foo and it all has the same type, allow x.foo to generate the obvious match
  * **Niko**: must be in the front, right?
  * **Patrick**: doesn't have to be
  * **Dave**: different cost model
  * **Niko**: I want to know that I'm not generating that match
  * **Patrick**: if we allowed that, we wouldn't need the common-field syntax, might be just enough to get us over the painful pattern
  * **Niko**: my opinion is, it's gonna be cited as a wart. I guarantee you. but maybe that's ok
  * **Paul**: wasn't the idea originally part of some grand plan that unified a bunch of things?
  * **Patrick**: kind of separate; idea was to unify enums and structs; common fields were off to the side
  * **Patrick**: wasn't happy with new syntax, or additional concept in the language
  * **Niko**: the whole problem is the syntax, right?
  * **Patrick**: also, new syntax, right? what is lower cognitive overhead?
  * **Graydon**: I think being explicit about this is a little bit better, even if it's a little janky syntax
  * **Graydon**: which thing is a wart?
  * **Niko**: (*dave missed it*)
  * **Graydon**: gonna be a common pattern, might as well institutionalize it
  * **Tim**: what's wrong with refactoring so that you have a containing record with nested enums?
  * **Patrick**: can't do refinements. can't refine A|B|C to just B
  * **Tim**: okay, sure
  * **Patrick**: we see this pattern a lot in Servo: passing nodes they know to be HTMLImageElement, then matching with a fail all over the place
  * **Graydon**: I think common fields is okay; we're already doing it to an extent; I'd kind of like this to work
  * **Graydon**: currently have a common field in every enum, 64 bits wide, would be awesome in the future to pack some extra values into spare bits

# Unify trait/impl syntax

  * **Patrick**: there's a bug on ability to define trait and impl in one go
  * **Patrick**: what should the syntax for that be?
  * **Sully**: do you really want to? when does that happen?
  * **Lindsey**: happens a lot
  * **Lindsey**: it's all over the compiler
  * **Niko**: anyone who wants to write using method syntax
  * **Lindsey**: really exploded after coherence. not sure why
  * **Sully**: you only need to write the trait if it's a base type, right?
  * **Patrick**: people love to define new methods on int that are only available in their crate
  * **Brian**: maybe that's a bad sign. now that we have coherence those all belong in the original crate
  * **Niko**: I think it's nice. I like to write code that way, I like method syntax
  * **Niko**: I don't think those methods belong anywhere but where they are
  * **Patrick**: question of whether there's a way to do this in one go; a lot of people think yes
  * **Patrick**: brian has some reservations about current syntax anyway; many type parameters leads to crazy declarations
  * **Patrick**: maybe these things are intertwined
  * **Brian**: it'd be nice if everyone was happy with trait syntax for 0.4, b/c there are some examples that are pretty nasty
  * **Patrick**: mostly with generics, right?
  * **Brian**: yup
  * **Graydon**: haven't written enough myself, can't really comment; worth doing via an RFC b/c it's gonna be a lot of syntactic comparisons
  * **Patrick**: sure
  * **Patrick**: that's all I have. someone else talk

# Macros and crate dependencies

  * **Graydon**: trying to reason about versioning, unifying cargo's and the compiler's approaches to versioning and dependencies
  * **Graydon**: one of the original justifications of separate crate files was a phase distinction; to find dependencies, would be nice not to have to macro-expand
  * **Graydon**: so one thing I would do is have shebang declare syntax extension dependencies
  * **Paul**: you're talking about user-defined macros, not built-in syntax extensions?
  * **Graydon**: no, talking about built-in extensions won't be built-in forever
  * **Dave**: I would avoid expanding into `extern` / `import` / `export` but not add a new declaration mechanism beyond the existing ones
  * **Eric**: bindgen is a good example where you'd like to generate extern declarations

# Default methods

  * **Graydon**: they work, thanks to Lindsey!

(applause)

  * **Lindsey**: working on next: default methods that refer to self, and then we *will* be able to start making code better
  * **Lindsey**: it's Eric's and my last week
  * **Lindsey**: if I don't get to document default methods in the tutorial before I go, I'll do it on the plane on the way home
  * **Paul**: giving my final intern preso tomorrow
  * **Tim**: do you know what time?
  * **Dave**: somewhere between 2 - 4, just like your cable repair person
  * **Tim**: that's be 8am to 8pm
  * **Graydon**: death to modes! yay
  * **Graydon**: tree is green again
