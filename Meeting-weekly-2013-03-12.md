# Agenda

- GC update (graydon)
- Amazon status (graydon)
- New contributor update (graydon)
- Triage plan (graydon)
- RFC responses: (graydon)
  - bool => Bool and make an enum
  - 78 column limit => 100 column
  - (there are zillions of them, we'll triage the rest before digging in)
- removal of base types (pcwalton)
- &'static => &'const? (pcwalton)
- overloaded operator autoderef (pcwalton)
- enum variant values? (pcwalton)
- global variables (pcwalton)
  
# Attending

- graydon, brson, jclements, pcwalton, nmatsakis, pnkfelix, azita, tjc, Jack Moffit

# GC Update

- G:  Sort Of Working GC but for One Last Bug
- G: Currently somewhat slower than RC, scan times are slow and imprecise
- G: But binaries are 20% smaller
- P: Does this improve trans running time?
- G: Not measured yet

# Amazon Status

- G: Down.  Snapshots and BSD are not working.

# New Contributor Update

- G: Adding new contributors.  They have full rights but the idea is to avoid making core language changes since those operate on consensus model.  But bugs can be closed, reviews done, etc.
- G: Reviewer list is on github under settings (Rust Push team).

# Triage Plan

- G: Interesting post about Firefox teams making strategic recommendations about triage
- G: Pre-divide bugs and send them out *in advance* to those attending the triage meeting
- e.g., 20 bugs per person, close those in advance if possible, otherwise have them in mind
- N: 20 different bugs per person, right?
- G: Exactly.  I will try to organize a meeting following this rough plan.
- Azita: this will take the place of the other triage we do?
- G: I hope so.  Certainly it will be more systematic than today.
- G: I will try to get that in a more community format since the more help the better.

# RFC Responses

## Boolean

- G: There are many RFCs but here are some very recent ones.
- G: Change boolean type to an enum rather than a built-in.
- G: On theory that...why not? 
- G: Implications: you can shadow true and false.
- G: I don't want to do this (as I said in the bug) but I'm willing to entertain strong opinions on the matter.
- B: Only reason that the situation bothers me is that `true`/`false` are keywords but `bool` is not. 
- P: Go has `bool`, `true`, and `false` as identifiers that are always in scope
- P: Our primitive type names in general are currently treated as identifiers that are always in scope. 
- P: Capitalizing `Bool` etc seem strange, as would be implied by the naming conventions
- P: There are legitimate reasons to keep it a primitive type (C compatibility) 
- P: But I think Brian's point is strange given that we have pre-declared identifiers like the type names already
- G: I think of `true` and `false` as literals and those cannot be rebound in general.  Granted, they are lexically similar to identifiers.  Like #t and #f in LISP.
- P: Can't you rebind those in LISP?
- G: What I mean is you cannot rebind 1 or 2 or "hello" so why true?
- N: Do we allow constants in patterns? I don't think we do today but I think we should
- G: I don't think we do today though we do allow enum variants
- N: ok ok
- P: I don't care too much about it.
- G: Nobody cares ok let's say no.

## 78 Column Limit

- G: I think I'm the only one who likes it
- F: No, I do too!
- G: Yes!
- G: My argument in favor is side-by-side editing with a two-column vertical display
- F: Yes, yes!
- P: Rightward drift becomes problematic, particularly in match
- P: We tried half-indent but then it didn't read well
- B: Also impls
- F: Can this be customizable on a per-file basis?
- G: It's just a tidy script
- P: There are some cases in resolve that are really hard
- N: Arguably sometimes those should be pulled into different functions
- N: 78 char limit does work against long names
- B: I was working on UV code, which is asynchronous
- B: you often want it to be in the same function
- B: I was spending time refactoring to make names shorter
- G: This turns into one of those aesthetic calls, let's just ask for votes
- G: 2 against... everyone else in favor?
- JC: Not sure
- *OK*

# Removal of base types (pcwalton)

- P: Most people probably do not know these exist
- P: For a `impl Type` form, you can write a pseudo-arbitrary type there
- P: e.g. `impl &Type`
- P: This was intended to work around lack of explicit self
- P: I would like to remove this feature, it doesn't seem useful and it offers More Than One Way To Do It
- G: So you could only do `impl NominalType` or `impl Trait for Type`?
- P: Yes
- F: Can you overload the self type per-method?
- P: No, not right now.
- F: Does this make the language less expressive?
- F: Like, today you could do "impl @Foo { fn foo(self); }" vs "impl &Foo { fn foo(self); }"
- P: Yes I'd like to remove that but you can still do it with traits
- G: Yes this applies only to inherent methods
- ... some discussion on inherent vs trait and how such overloading is still permitted...

# Rename 'static to 'const?
- P: Let's rename the static region to const since it refers only to constants and it would let us remove the `static` keyword entirely.  
- N: +1. I think Graydon wanted this name originally.
- B: Don't we also want to stop overloading const?
- P: The const-as-supertype-of-mut-etc will go away
- N: I think all the remaining meanings would actually be compile-time-constants
- G: It should mean "read-only memory segment", I think that's nice
- P: `Const` trait will hopefully go away
- P: So then `const foo = ...` and `'const`
- G: What's up with the Const trait?
- P: Without mut fields it's not that necessary
- N: I'm not entirely sure about that but there are alternatives in any case

# Overloaded operator auto-deref

- P: John for some reason keeps banging his head against this
- P: Currently auto-deref works for overloaded operators but only for the LHS
- P: So if you do "a == b" it'll autoderef `a` but not `b` and you 
- N: We should not do any of that on overloaded binary operators
- P: Except maybe `[]`
- G: Keep it for `[]` but not the binary operators  like `+` `-` etc
- P: OK, so remove for everything but index.

# Enum variant values

- P: I've kind of softened my view but I wanted to raise it up to get a decision
- P: Right now you can supply explicit discriminant values for an enum
- P: And you can access them with `as`
- P: I originally thought this was completely useless, and it is indeed not as useful as one might like since you cannot go in the reverse direction
- P: But then C++11 enum classes share the same restriction
- P: And people use this feature all the time
- P: I still feel like the right thing is to have a macro that expands etc
- P: But I thought I would ask what other people think
- B: Can we go further and remove C-like enums?
- P: What does that mean?  What would be left?
- B: Specifying the discriminant?
- P: That's what I'm proposing removing
- P: Regarding C compatibility of enums: you cannot achieve that because the C standard 
- N: You can still do what people "do in practice"
- G: The C standard does not specify a calling convention either
- P: ABIs don't specify the size of enums either
- G: Regardless there are many
- N: Why can't we just say that you can apply an attribute to the enum that says what kind of int to use, and otherwise we use some default rules?
- G: What will happen if we don't have this is that if there is an API with an enum, people will pick an arbitary integer to use.  This feels very similar to figuring out constants etc.
- jld: The x86_64 SVR4 ABI (in a footnote, admittedly) does seem to specify the  sizes for enums: the first of `int`, `unsigned`, `long`, or `unsigned  long` that works.
- P: I'm fine with saying we'll do what C++11 does but no more, which is basically what we do today.
- tjc: Haskell uses a typeclass with to_enum and from_enum...
- P: ...but this prevents us from using it in constants, which is basically the reason that `as` exists
- N: I think I'm fine with how it is today
- G: I think it's incomplete, particularly with respect to constant evaluation
- N: I meant more like "the platonic ideal" of how it's implemented today
- N: Reverse direction strikes me as a simple syntax extension
- P: OK, I'm fine with it due to C++11 precedent

# Global variables

- P: I think they're inevitable and we've already snuck them in with constants in `extern mod`
- G: Don't forget, we got inline assembly this week.
- P: We bikeshedded syntax a bit, current winner: `unsafe mut x: Type = ...;`
- JC: Global to ... what?
- P: Scoping follows the same rule but they are global to the program
- P: And you can put them in an extern mod to access global variables like `screen` from `curses`
- G: Yeah, they exist in C libraries and there's not much we can do about it
- P: Plus scheduler needs it
- B: Need a number of atomic locals and mutexes in various places
- N: Is anyone opposed?
- G: To me this comes down to unsafe: as Rafael said, if you don't provide people the ability to do something that you can do in C, then they will just link to C
- N: I'm in favor 
- B: Not that crazy about the syntax
- P: That was the current winner of our bikeshed
- G: Clearly `const mut`!
- G: unsafe { let mut x : T = ... ; }  no good?
- N: What is wrong with `let`?
- P: Well, currently `let` is only used with locals, and `let` introduces a new scope whereas globals would be in scope everywhere.
- JC: Can they have arbitrary initialization?
- G: No, constants only, no constructors with this feature
- N: I think `let` as a global can differ from `let` as a local
- P: I find it strange, a common question is what's the difference between `let x` and `const x`?  One of the biggest differences is the scoping of `const x`, and of course `const` can only be compile-time constants
- N: I think the big difference is that `const` is a compile-time constant 
- G: Worth mentioning that *declaring* is not unsafe, just accessing it
- G: So the declaration doesn't need to be annotated with unsafe
- JC: So you would have an unsafe block to access them?
- G: Definitely.
- P: A little different than pointers because you have to take an action to cause unsafety with a *ptr, whereas these would be lvalues
- G: Yes any expression 
- N: I just want to point out that if you take the address of a global variable, you would get &'const mut as a lifetime (&'static mut today)
- B: Maybe we should keep `static` and just call the variable declaration `static`?
- P: I'd rather just add a `global` keyword, I'm ok with `const mut` honestly
- *general grimaces*
- G: Following the C tradition of overloading the word static, why don't we go the opposite direction and just get rid of `const` and call it `static`:
    - `static x: T = ...`
    - `static mut x: T = ...`
    - `&'static`
- N: That's.... logical.
- F: Is that different from how C uses static?
- N: I think that's pretty consistent, actually.  A `static mut` inside a function you get something identical to a C static.
- P: Except for the RHS.
- JC: That's... seems so ugly to have a global inside a variable
- N: What's the difference?
- P: It may be better since it controls the scope
- G: I concur it's ugly but we do it
- N: From chat: " If you add  globals so you can bring globals from C libraries into scope, and you  call it static, I'm gonna be a confused C programmer who doesn't  understand why static suddenly doesn't mean file-scope anymore like in C  :)"
- N: Though they are *module-local* by default
- P: Yes you'd have to write `pub static` 
- G: I think that's actually where my vote goes
- P: I'm fine with it
- G: I'm going to go and cry since I love the word `const` but ...
- G: ... we would never live down `const mut`
- N: I think it's ok, though I guess we can stew on this a little
- G: Maybe run it by the mailing list (bike shed wheee!)
- P: Yeah why not.
- G: Occasionally someone comes up with something amazing we didn't think of
- P: Yes, the lifetime syntax was quite fruitful
- N concurs
