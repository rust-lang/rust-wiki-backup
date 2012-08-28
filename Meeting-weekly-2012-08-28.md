# Agenda

- Overloaded `extern` (pcwalton)
- Universal equality (pcwalton)
- Method call syntax (nmatsakis)
- Enum refinements (nmatsakis)
- Release status (brson)
- Compressing metadata (graydon)
- Intern talks this week
  - eds on Thursday at 12pm (but show up 20 minutes early in case previous talks run quickly)    

# Overloaded `extern`

- pcwalton: There are four meanings for extern
- pcwalton: Basically what I said in my e-mail
- pcwalton: 
    - `extern mod foo {}` vs `extern mod foo;`
- graydon: I want to try and minimize the extent to which people think of linking C and Rust as radically different activities
- pcwalton: A (mandatory) attribute that specifies the ABI would go a long way
    - in particular, on the `{` form
- brson: I think it's incredibly confusing that `extern fn` means something else...
- pcwalton: ...that's the second part.  Currently `extern fn` in the type means a function that closes over nothing.
- nmatsakis: it should just not be a type.
- pcwalton: what we used to call crust fns ... those have type `*u8` but this prevents us from supporting monomorphization and so forth
- pcwalton: I propose making `extern fn` mean that
- nmatsakis: A useful thing would be a C function pointer
- brson: to call it from Rust, you need to put an ABI in the type
- pcwalton: can we make them coercable?
- nmatsakis: yes, but you have to generate a thunk
- brson: if all we had were external decls of functions, we wouldn't need the whole native mod business
- pcwalton: you do need a place to write the linkage details
- graydon: I think the extern mod syntax is worth keeping around just to prevent you from writing extern all the time and to help with the linkage information, but it shouldn't carry a lot of other semantic details
- pcwalton: especially in systems with 2-level namespaces, it's important to have the name of the library somehow associated with the fn
- graydon: this comes into sharp relief in windows
- graydon: in linux you can do all kinds of things, but in windows you have to declare it as coming from a particular library
- graydon: it would be enough to say that every external function has to have a library attribute, but we could just say that it's taken from the innermost enclosing attribute
- nmatsakis: so how do people feel about a type like `extern "abi" fn()` to mean just a function pointer?
- *general agreement*
- pcwalton: then we can get rid of crust fns altogether
- *general confusion*
- pcwalton: yes, if we permit coersion from rust to C...
- nmatsakis: ...only possible for bare fns (not arbitrary fn values)...
- pcwalton: ...yes, only in that case
- pcwalton: fns that you wish to use as a callback would just be plain old fn items, but if you use them where an `extern "abi" fn` is required, it would generate the wrapper at that time
- brson: that's a lot of work to do under the hood
- graydon: how does that compare to what happens now?
- nmatsakis: basically
- brson: it is an implicit performance hit...
- nmatsakis: why? no worse than today, right?
- brson: ...only thing that's worse is that you never create a fn that's slow
- nmatsakis/graydon: it feels like the cost of stack switching is something we should solve anyhow
- graydon: in particular, the cost of calling Rust->C is the same as C->Rust, and that ain't going away.  Maybe stack switching isn't the right approach.
- brson: another thing I was wondering: is `extern "abi" fn` expressive enough to cover the various cases?
- nmatsakis: not enough for C++ member functions, but nobody uses those, but to handle name mangling we'd have to make extern fns invariant with respect to argument types (which is fine)
- graydon: neither is enough for varargs
- graydon: same thing will come up with linking to fortran with its array types
- pcwalton: if we ever do want to support linking against C++, the way D does it is instructive: they have a simple subset of well-defined C++ behaviors that permit ABI compatibility
- brson: so if we type `extern mod` do we have to write `extern fn` inside
- pcwalton: it seems like basically the same thing we do today 
- brson: I think it's better
- graydon: we could move over to an extern block, like in C++, so that it doesn't introduce a new namespace
- brson: I like that better, too
- brson: ok, so if we write `extern {...}` everything inside of it is an external declaration
- nmatsakis: we should pay attention to collect the stack-switching code in trans

# Universal Equality

- pcwalton: I made some experiments at getting rid of universal ord, very successful
- pcwalton: Universal ord (<, >) is pretty much never used
- pcwalton: I removed universal "<" but I think it's never used
- brson: What would be the plan for "<=" 
- graydon: I saw you working on this and it harkened back to some very old questions
- graydon: good defaults for cmp, memcmp (and ignoring holes), deep/shallow equality, tc
- graydon: I'm increasingly thinking we made the wrong decision here in terms of taking control from the user
- graydon: maybe the best thing is to implement the basics and let the users override as they want to
- pcwalton: I implemented it for many types (option, [], pairs)
- graydon: you talked about an automatic deriver...
- graydon: that doesn't sound too loopy to me
- pcwalton: I was actually thinking that you could do it sort of like Haskell does it
- pcwalton: the only derivers we actually *need* are enum/struct
- nmatsakis: this is a nice convenience
- graydon: ...shades of Scrap Your Boilerplate...
- pcwalton: the nice thing is that we can bake most of it into core
- nmatsakis: as long as you don't mind that it only works for tuples up to some size
- pcwalton: are people ok with me continuing work on this?
- nmatsakis: have you tried universal equality?
- pcwalton: working through it, it's mostly mechanic transformations
- nmatsakis: I've had bugs on universal equality from time to time so I'm not a big fan
- graydon: is this rubbing anyone the wrong way?  only heard from 3 of us
- tjc: it's fine by me, same as haskell, I don't feel I rely on universal eq a lot
- graydon: particularly if tuples work

# Method call syntax

- nmatsakis: Do we want `a.b()` to mean as either a field-with-closure-type or a method?  I'm gonna be refactoring the AST and if we wanted to change it'd be a convenient time
- graydon: I'm ok with that, now we have real methods, a record filled with closures seems like it won't be common
- nmatsakis: a record w/ closures kind of seems like an anti-pattern to me
- graydon: it deletes the ambiguity 
- nmatsakis: brson, tjc?
- brson: I kind of disagree that it won't be used a lot, I can imagine some scenarios, but I still think we should do it

# Enum refinements

- nmatsakis: Just wanting to suggest people read my [blog post][bp] so we can discuss it later:
[bp]: http://smallcultfollowing.com/babysteps/blog/2012/08/24/datasort-refinements/

# Release status

- brson: 83 issues open for 0.4 and 2 weeks to go
- graydon: triage will be necessary, shall we spend an hour or two together going through it?
- tjc/brson: yes that could be useful
- graydon: honestly this is something that most of the rest of Mozilla does regularly, we ought to do as well, it would probably help us with prioritiziation
- graydon: is there anything more you wanted to discuss?

# Compressing metadata

- graydon: our metadata tables are enormous.  Part of it was a debugging flag that Niko left on (ed: d'oh!)
- graydon: nonetheless our metadata section is many times larger than our code
- nmatsakis: could be partly because of generic fns whose code may not be generated
- graydon: in any case, it's not necessarily bad that it's big, we could get easy benefits by running some compression on the data
- nmatsakis: that would prevent us from having random access
- graydon: ...which we don't do anyhow, right now
