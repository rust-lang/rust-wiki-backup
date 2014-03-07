Many changes can be reviewed after-the-fact, or via bugs or the usual code-review mechanisms of github.

Some changes are "substantial", and we ask that these be put through a bit of a design process and produce a consensus among the [core team](https://github.com/mozilla/rust/wiki/Note-core-team).

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

## What the process is

The process is still pretty lightweight: 

  - **Indicate you have an RFC in one of the obvious places below**, so the core group will consider it.
  - **Give the RFC some time** to ripen and collect feedback, at least a week.
  - **Acquire consensus** from everyone in that group. Consensus is the only vague part, but at least look for an explicit "yes" by more than half the group and **no outstanding unaddressed concerns** from anyone else, nor any reason to believe anyone else might have any. We're not always paying attention to everything, so you have to use some judgment about this part.

### "Obvious places?"

The following are places where RFC-requiring bugs will be seen:

  - Make a bug tagged `[B-RFC]` that explains things in detail. Make a proposal page in the wiki if the proposal is much longer than a few paragraphs, or you expect to be revising it. Wiki pages are more easily editable in place than bugs.
  - Post an email with `[RFC]` in the title, again with a proposal page if the proposal has much size or you expect much revision.
  - Attend one of our in-person meetings and discuss it there.

For our part, we will try to run through the RFC-tagged bugs at our weekly meetings, to make sure that if there's consensus among the attendees, an explicit expression of it gets made. If a bug is not RFC-worthy, someone may remove the `[B-RFC]` tag as part of bug triage. Using RFC-tagged bugs are best since they naturally queue up.

### Help this is all too informal!

  - Use your judgment.
  - Err on the side of over-communication.
  - Apologize and revert the code if you make a mistake.
