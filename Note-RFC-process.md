Many changes can be reviewed after-the-fact, or via bugs or the usual code-review mechanisms of github.

Some changes are "substantial", and we ask that these be put through a bit of a design process and produce a consensus among the core developers (those with administrative access to the mozilla/rust repository).

## When you need to follow this process

You need to follow this process if you're going to make a "substantial change" to any of the following:

  - The file `src/comp/syntax/ast.rs`
  - The reference manual, tutorial, or man page.
  - The development environment (.gitmodules, cargo, dependencies, Makefiles, configure script)

### "Substantial?"

Where we define "substantial change" as any change _excluding_:

  - Rephrasing, reorganizing, refactoring, or otherwise "changing shape does not change meaning".
  - Additions that strictly improve objective, numerical quality criteria (warning removal, speedup, better 
    platform coverage, more parallelism, trap more errors, etc.)
  - Additions only likely to be _noticed by_ other developers-of-rust, invisible to users-of-rust.

## What the process is for such substantial changes

The process is still pretty lightweight: you need to convince yourself that your change has been 

  - Put in an obvious place to be seen by everyone in the core group (currently: andreasgal, BrendanEich, brson, catamorphism, dherman, erickt, graydon, jdm, jruderman, lht, marijnh, nikomatsakis, pcwalton)
  - **Agreed-on** by everyone in that group, or at least explicitly agreed-on by a good number (more than 4?) and **leaving no outstanding unaddressed concerns**, or the impression that anyone else might have any.

For our part, we will try to run through the RFC-tagged bugs weekly to make sure that if there's consensus among the attendees, an explicit expression of it gets made.

### "Convince myself?"

The following are some effective ways to convince yourself:

  - Make a bug tagged `[B-RFC]` and wait a week. Ensure that several developers from the core group have said "yes" and there are no outstanding, unaddressed concerns raised.
  - Post an email with `[RFC]` in the title and wait a week. Same standard of conviction: several core developers have agreed and there are no outstanding, unaddressed concerns raised.
  - Attend one of our in-person meetings and get the same level of verbal consensus.

### Help this is all too informal!

  - Use your judgment.
  - Err on the side of over-communication.
  - Apologize and revert the code if you make a mistake.
