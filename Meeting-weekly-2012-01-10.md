## Attending

Brian, Dave, Graydon, Jesse, Marijn, Niko, Patrick

## Remaining issues for 0.1 release

### [Issue #1454](../issues/1454)

* Niko: we can spawn tasks, joinable tasks, and connected tasks; three different API's, which could be improved
* Graydon: all library things will be junky for a while; that's ok for first release
* Niko: syntax for various functions needs work; what should we call bare functions and blocks?
  * lambda --> `fn@`
  * block --> `fn~`
  * or block --> `fn` but then what do we call bare functions?
* Marijn: could you update the tutorial when you make this change?
* Niko: yes, once we decide on syntax
* Jesse: call blocks "stack fns?"
* Niko: yeah, I like that, or stack closure
* Graydon: terminology-wise that's fine; in the syntax, I'm ok calling bare functions `native`; doesn't enter into programs too often
* Niko: sure. could also change the representation to a naked pointer but I don't want to risk the bugs before first release
* Graydon: yeah, rep has tons of grotty stuff in C ABI, not likely to work easily
* Niko: OK, I'll make the change this morning

### [Issue #1448](../issues/1448)

* Graydon: not a blocker
* Niko: it's confusing
* Graydon: yeah, but not a problem for 0.1 release

### [Issue #1446](../issues/1446), [#1445](../issues/1445)

* Graydon: needed for release

### [Issue #1430](../issues/1430)

* Graydon: no opinion
* Niko: I never know when to use `;` and when to use `,` -- better to make these changes now if possible
* Graydon: I'm fine with it
* Patrick: I may do this one in between Fennec compiles

### [Issue #1428](../issues/1428)

* Patrick: I'm almost ready to give up on this question
* Dave: I like `enum`
* Patrick: what about `newtype` analog?
* Niko: not a big deal
* All: `enum` is good

### [Issue #1427](../issues/1427)

* Tim: I'm doing this

### [Issue #1426](../issues/1426)

* Graydon: not blocking

### [Issue #1425](../issues/1425)

* Graydon: not blocking
* Niko: non-trivial
* Patrick: I started, won't finish in time

### [Issue #1381](../issues/1381)

* Graydon: easy, along with release notes

### [Issue #1343](../issues/1343)

* Graydon: not cool
* Niko: it's a problem; can we comment out that code?
* Brian: just doesn't work to build with debug on 32-bit
* Niko: it used to work
* Graydon: feels like a release note
* Patrick: force it off and warn when compiling with `-g`?
* Brian: I'll do that

### [Issue #1078](../issues/1078)

* Niko: this is currently in C++, has leaks when failing during unwind etc, but shouldn't block release
* Niko: I'll update

### [Issue #1011](../issues/1011)

* Graydon: there's a supposed solution?
* Brian: two Windows features that supposedly provide rpath functionality; could add something before `main()` to set up paths; seems like a hack
* Graydon: but might work! alternative is inject something in installer to update `PATH`
* Dave: sounds brittle
* Graydon: and users don't like programs mucking with their `PATH`
* Brian: I'll look into it

### [Issue #427](../issues/427)

* Niko: unfinished; has holes; task system needs work
* Graydon: type classes?
* Marijn: not sure it's ready for broadcasting
* Graydon: I'm working on converting manual to markdown; can we merge them?
* Dave: I kinda like having them separate; manual is official reference, whereas tutorial is a starting point
* Patrick: yeah
* Marijn: who styled the lib docs? could use restyling -- do now?
* Graydon: I'd like to escalate getting rust-doc formatting the way we want on the web; everyone will start using it and it will create a standard style
* Brian: you'll do that?
* Graydon: yeah, I'll look into it
* Dave: would it make sense not to include rust-doc in release yet?
* Graydon: no, just needs a week or so of cleanup

### [Issue #32](../issues/32)

* Brian: LLVM patches have gone through 2 rounds of review; should be accepted today and then we can go back to LLVM trunk
* Brian: we have huge red ones but we can punt on that for 0.1
* Graydon: great, so we can close this issue in another day or two

## Release

* Patrick: I propose release in a week
* Graydon: well, I'm fighting makefiles! hope we can make that
* Niko: one thing not in list: stuff from fuzzer
* Graydon: I'm okay releasing 0.1 with bugginess
* Graydon: Patrick -- syntax changes?
* Patrick: yeah, I can do those

## Work week

* Dave: work week: Berlin? Feb?
* Niko: I have some constraints
* Dave: I'll collect constraints
* Andreas: Marijn -- know any good Berlin hotels we can use?
* Marijn: no, but I'll look

## Object system

* Patrick: motivation for classes
  * I wrote up a proposal a while back
  * typeclasses subsume interfaces
  * when you have a first-class value w/ typeclass as type, you get a structural object
  * what we have in classes that pure typeclasses don't is private data
  * that's why I'm in favor of having class aspect of original proposal
  * nominal records, private fields, constructor
  * can cast into typeclass with subset of methods
* Dave: note that "interfaces" in Haskell are bounded existentials
* Niko: interfaces like closures; worth sigils for sendability?
* Graydon: sounds like stuff for further down the road
* Marijn: you can get privacy with old system using hidden types exported as abstract types from modules
* Graydon: I think we could remove our old object system
* Marijn: for 0.1 leave t
* Dave: I don't mind leaving it and shipping
* Niko: I'd rather have a more stable one; currently buggy but don't want new bugginess
* Graydon: one other point: `self`-calls and overriding
* Marijn: overriding doesn't exist in the abstract types approach
* Dave: don't want to risk new bugginess at 11th hour
* Graydon: I'll take out of docs, say it's deprecated

## Syntax

* Dave: syntax point: I heard about Go's cast syntax and it sounds sweet

## Admin, release

* Graydon: anyone have make-or-break packaging issues? raise now
* Patrick: Mac & Linux users comfortable with `git clone` -- Windows is the main issue
* Dave: can we transfer to `github/mozilla/rust`?
* Jesse: bunch of broken links in wiki, Google is linking to 404's
* Dave: I'll give Graydon admin rights to mozilla github account
* Graydon: branching/tagging model? git flow?
* Brian: could we prohibit landing on master for a few mnoths?
* Graydon: we'll continue this conversation by email
