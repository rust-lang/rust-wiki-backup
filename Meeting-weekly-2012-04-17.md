## Attending

Dave, Patrick, Niko, Graydon, Marijn, Tim, Brian

## Alt syntax

  * **Patrick**: people don't like my if syntax, which is good; Florian found a gotcha on list: one-line if forgetting braces; but people did like fat-arrow alt, so I propose that
  * **Niko**: in favor
  * **Graydon**: lone dissenting voice, but OK, I'll lose on this one :)
  * **Niko**: don't know if it should be single or fat arrow; two arrows seems too much
  * **Patrick**: I slightly prefer fat arrow
  * **Tim**: Graydon, why don't you like it?
  * **Graydon**: reminds me too much of ML; the syntax of ML seems to scare people off
  * **Patrick**: Niko said helps you distinguish between between patterns and expressions when you're looking at code; deeply-nested alts are hard to read, because patterns don't stand out
  * **Brian**: agree
  * **Niko**: agree
  * **Niko**: been wanting to propose case, but fat arrow serves same purpose
  * **Brian**: always arrows / always no arrows, vs allowing going back and forth?
  * **Niko**: always arrows / always no arrows
  * **Patrick**: agreed; better for pretty-printing, better for reading

## Clang integration

  * **Graydon**: one big work unit on my plate for this release: start integrating clang; take the dependency, try knowing something about C natively; clang has its own library for executing cmd-line tool chains; migration path for integrating bindgen, directly reflecting on C code
  * **Patrick**: totally ok
  * **Brian**: thumbs up
  * **Dave**: how surfaced to user?
  * **Graydon**: initially, nothing; expansion of LLVM library that rustc can call; allows us to generate C code, do stuff with all sorts of additional kinds of dependencies like libraries
  * **Marijn**: any chance of convincing clang folks to separate these things?
  * **Graydon**: set of bugs addressed by this cover a bunch of C integration issues; gives us a broader and broader set of C integration tricks we can pull, like scanning header files and generating bindings
  * **Dave**: that's such a big deal for an FFI
  * **Graydon**: could build small C shims and compile them usefully; don't want to grow entire command-line of a C compiler, but ability to process C when it's useful to us
  * **Marijn**: viable to bind C++ code, too?
  * **Graydon**: some possibility there
  * **Dave**: my only concern would be if the *language* started depending on it in a way that forced Rust to be tied to LLVM permanently
  * **Graydon**: eventually it'll probably be more and more useful to have a formal relationship to C, just as C++ does, but that's a ways off
  * **Dave**: agree

## Lint changes

  * **Graydon**: just overhauled lint pass: now a unified way of handling all possible warnings/errors; any warning or error that is not essential should be registered with lint pass; unified command-line handling (surfaces a -W flag for each registered warning); also allows using a scoped `#[warn]` attribute in code
  * **Graydon**: doing something Niko requested: once you add a new feature to compiler, can add a lint check to warn on uses of the old feature
  * **Graydon**: one subtle issue, current disagreement between Brian and me: tension between who overrides who: attribute vs command-line flag; inner attributes override outer attributes; command-line currently sets defaults for outermost layer; matches the way C compilers do their command-line warnings vs pragmas (e.g., gcc, msvc) -- weird counterintuitive effect that command-line can be overridden; Brian thought it might be useful to allow command-line to override
  * **Marijn**: you might have a corner of your code that *needs* to use something; maybe two different approaches available on command-line?
  * **Brian**: that'd be fine
  * **Niko**: I'm fine with both, but would be better to have these in .rc files
  * **Brian**: if you can do it either way at command-line, should also be able to do it either way from attributes
  * **Graydon**: I don't like the current syntax; "warn noctypes" sounds like the opposite of what you mean
  * **Patrick**: I like nowarn
  * **Brian**: can Rust codebase turn on all errors? I don't like having them and not using them; if they're gonna exist, we should be the people that use them all -- we should be the most strict
  * **Graydon**: I agree, but I think we can acquire things that are special purpose, like no-unsafe -- but we need unsafe
  * **Brian**: I just think we should turn on the maximum warnings
  * **Graydon**: yeah, and we can test out all the warnings and make sure they're useful

## Labels

  * **Graydon**: work on Servo currently blocked on a particular bug -- can everyone vote on syntax? issue: labelled break/continue; issue 2216
  * **Graydon**: no overhead version looks like crap; one token lookahead looks a bit better
  * **Niko**: hadn't seen this
  * **Graydon**: needed for generating HTML5 parser
  * **Dave**: are we talking about break/continue from local procedure only, or also within nested downard block functions?
  * **Niko**: local for now, but probably for completeness want to allow it in for-loops as well
  * **Graydon**: will that break our current boolean-flag implementation?
  * **Niko**: no, can be done

## Crate files

  * **Graydon**: remove distinction between crate files and non-crate files? our users keep stumbling on this
  * **Graydon**: if you just think of crate file as root of DAG, make sure compiler doesn't get stuck in a loop, we still have same compilation artifact...
  * **Graydon**: feels like a loss, but somewhat flexible
  * **Brian**: I've come up with one way this ends up being pretty weird: in .rc we can specify external implementation of module via path; can do that for directories with companion modules; mod { ... } *and* an external file that defines *also* what's in that mod; they're merged, but not much an .rc file can do; if we eliminate .rc and leave path attribute -- which we have to -- we'll end up merging the entire contents of two modules for every module that might have an external source
  * **Graydon**: path attribute should probably change... 
  * **Brian**: not even clear what a directory mod is; right now it's a mod with a body that lives in an .rc file; syntactically indistinguishable from regular mods, except that it lives in a certain kind of file; if we merge the two, nothing to say "this is actually represented as a file in filesystem"
  * **Niko**: couldn't we just not declare the modules up-front? just use what's in filesystem, or declare pub or something?
  * **Brian**: if you can load a file from filesystem as a mod, won't have to declare what they are; like, if it's a mod without a body, comes from filesystem
  * **Niko**: seems like if you get rid of an .rc file, there'd be no central place
  * **Brian**: you'd declare all your child mods, and they'd be responsible for their children
  * **Patrick**: didn't get that; we had conditional compilation in .rc file; if we automatically pull linux on mac will get compiled
  * **Niko**: conditional compilation flags
  * **Graydon**: do it at place where you make the reference
  * **Niko**: not quite what I had in mind, but makes sense; I was just thinking scan the filesystem
  * **Graydon**: no, I'm not okay with that
  * **Patrick**: very Java-like
  * **Niko**: and soooo convenient...
  * **Graydon**: I want you to specify which modules and why, since you can do conditional
  * **Graydon**: just wanted everyone's attention on this

## Build logic

  * **Graydon**: migrate all build logic into a Rust program?
  * **Brian**: <pumps fist in solidarity>
  * **Graydon**: configure script and makefile check platform, download appropriate binary, and run it, and do nothing else
  * **Graydon**: our build can't go anywhere unless it downloads a binary anyway
  * **Graydon**: delicate, hard-to-debug dependency on platform; don't feel we're benefitting a lot from make
  * **Graydon**: we'd lose the dependency tracking and parallel build functionality of make
  * **Graydon**: when go did 1.0 release, they did something pretty: shipped top-level command called go that is a command-line wrapper to all their tools
  * **Graydon**: that's actually a nice discoverability feature for command-line user; we're already shipping several tools
  * **Dave**: I'm a fan
  * **Niko**: all the rage these days
  * **Patrick**: do we want to do it the way git does it? git foo is an alias for git-foo; lets you pull up man page easily with hyphen
  * **Niko**: so easy to extend
  * **Patrick**: yeah, just add rust-clam to /usr/bin and suddenly rust clam works
  * **Graydon**: then cargo might change name, but I wouldn't lose sleep over it
  * **Dave**: you can still call it cargo, just not the command name anymore
  * **Graydon**: all in favor of exploring this? <everyone>

## Vector changes

  * **Graydon**: new vector code in rustc: not usurped or colliding with existing vector code; one creates old version of vectors; one creates uniquely-owned vectors, two others are fixed-size and slice; starting to work; string versions work, vector not quite yet
  * **Graydon**: you can allocate on the stack now, can pass them around, if they're fixed-size and constant, don't cost; will figure out how to allocate constant vectors in constant region Niko has created
  * **Graydon**: there's a borrowing thing Niko and I have been working on
  * **Graydon**: will probably end up writing functions in the format we do now with funargs: will by default take most constrained version, which becomes the cheapest thing to pass around -- should be quite nice and speed up a lot of our vector code
  * **Graydon**: one curious wrinkle I hadn't predicted: vector addition doesn't really make sense anymore, because you have to figure out the allocation approach for the resulting vector; vector addition would always result in either shared or unique, but I don't know which to choose
  * **Niko**: we have a soundness problem re: constness: if you add const and mut, is it mut or const? right now we take LHS type; this is not a complete answer, but ... seems like a related issue
  * **Graydon**: possibly, yeah. part of me thinks maybe it's a fine time to remove vector addition using + and add libs
  * **Niko**: almost every use of + in our code is actually map; might not be so painful as you might think to remove it
  * **Patrick**: yeah. you could define it only on @ vectors
  * **Niko**: could require them to both be @ or both ~ and then the result is clear
  * **Brian**: I like that
  * **Graydon**: ok. I'll probably punt for now; maybe just do that; we might have to reconsider in the future. just think about it
  * **Marijn**: I'd say put it off, since it's not so useful and important anyway, and the new vectors working is more important

## Resolve rewrite

  * **Marijn**: resolve rewrite that ended up on my shoulders: I have not really thought much about public/private schemes and don't have strong opinions; anyone who has/does, please write an email to the list so I can see what people are thinking; haven't seen what people are thinking so it's hard for me to start implementing
  * **Patrick**: Niko had some thoughts
  * **Niko**: I had a proposal; Brian has convinced me maybe it's not the best idea; slowly persuaded towards simpler: public = visible from outisde your module, default = not visible from outside your module
  * **Patrick**: question is whether main has to be public
  * **Niko**: I think it can be private
  * **Brian**: agreed
  * **Marijn**: alternatives: everything is public if you don't export anything
  * **Patrick**: Dave doesn't like that
  * **Dave**: I'd prefer a modifier on the module for the convenience, so it is still clear from a glance that everything is exported
  * **Graydon**: want to avoid boilerplate
  * **Patrick**: having main allowed to be private is nice
  * **Niko**: makes "hello world" not so silly-looking
  * **Niko**: for most modules, most functions are helpers
  * **Niko**: maybe just a `#[pub]` attribute on module
  * **Graydon**: pub *; is not so bad
  * **Niko**: sure, that works too
  * **Dave**: I'm not going to stand in anyone's way
  * **Graydon**: someone should write this up so Marijn's unblocked
  * **Patrick**: private by default; pub on individual items; pub *; at top
  * **Brian**: imports and re-exports...
  * **Niko**: pub import looks pretty good
  * **Niko**: somebody should write it up
  * **Patrick**: I'm writing it up now
  * **Graydon**: Patrick: can you include in the write-up a change in keywords? import is probably the wrong word, and `use` is better; I think `use` has a long and colorful history of being misunderstood in our system; when you link to external thing, you should use `link` :-)
  * **Niko**: mount?
  * **Brian**: agree, we wanna put it right here, in this spot
  * **Graydon**: but corresponds to linker
  * **Niko**: maybe it's pushing the analogy to filesystem too far
  * **Patrick**: enums?
  * **Niko**: if it's a public enum, the variants are public by default; priv in front lets you hide them
  * **Niko**: I think we can use this approach for objects too, right?
  * **Graydon**: my general understanding: any time you have a default and a way to override, need a way to get it back again
  * **Niko**: yeah, that's what priv is for
  * **Brian**: classes are the other way: pub class, members are still private
  * **Dave**: if there are too many different defaults in too many settings, users might start getting confused, but you can change those policies based on user feedback
