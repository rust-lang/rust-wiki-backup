# Possible 0.5 priorities

### Complete traits to unblock development of standard libraries (nmatsakis)

I personally think a good overarching goal for 0.5 would be to "unblock" the development of strong standard libraries.  It is my opinion that the single biggest blocker here is our incomplete trait implementation.  In particular:

- there are bugs around explicit self, not all of which have issues associated with them;
- default methods and trait inheritance don't really work;
- we need a deriving-like solution for implementing repetitive traits like Eq, IterBytes, Serialize, and so forth;
- some lingering decisions on precisely what kinds of impls will be allowed (I have an in-progress blog pending on this post...) to avoid theoretical quandries when resolving;
- @Trait, &Trait, and ~Trait do not really work.

Fixing those things, primarily the first three, strikes me as very high priority.  It would allow us to define nice trait hierarchies similar to the ones found in Haskell/Clojure (Eq, Ord, Hash, etc) which form the foundation of a strong library design.

### Miscellaneous regions bugs (nmatsakis)

There are various bugs in the region type system implementation.  I will go through and collect some issue numbers later.  They ought to be fixed.  


### Condition-handler system (graydon)

It's difficult to write libraries presently that have "recoverable" failure modes. We've long talked about a pattern for this using TLS. We gained TLS in 0.4. We should give this a try in 0.5, it'll help a lot in writing reasonable libraries.

### Start on new driver, build and command-line tooling (graydon, brson)

Probably won't be time to finish this, but a new driver, as well as a maintainer-mode tool for bootstrapping the compiler and doing tasks currently done via 'make' (rather than using the makefile) should at least get started. Also probably involves adding a top-level `rust` command and renaming subtools a bit.

This should integrate with cargo, and take into account Servo's build requirements, because Servo's build system is a nightmare and we want to move completely to cargo, for the good of the entire Rust ecosystem.

### Performance work (graydon)

The build time is a regular source of complaint during development, and it limits a lot of what we can get done in a day. We should spend some energy on speeding this cycle up and making sure everyone's able to build as quickly and in-parallel as possible.

### Build automation, code review and integration throughput (graydon)

Switching our build system over to buildbot (now reviewed and in testing) will be good here. Bringing more and faster build slaves into production. And teaching the automation to do integration builds for us rather than bouncing off incoming.

We should also experiment with some more-structured code review, as our velocity slows and we start focusing on quality and performance issues in this and subsequent releases.

### Remove obsolete features (brson)

Including modes, export, structural Records, #-macros, [mut], assert/log/fail, <-, <->

### Upgrade and unblock uv on x86 (brson)

Our uv build is very, very far behind upstream, and simply doesn't work on 32-bit x86 because of rustc bugs.

### Cargo improvement and continuous integration (brson)

Keeping out-of-tree code up to date has become a very labor-intensive process, and our support for out-of-tree libraries is bad. I very much want to set up a continuous integration server for the entire community. Doing this completely in Rust helps uv, cargo, and servo.

### Tree cleanup and reorganization (brson)

There are a few things I would like to do to clean up the build, including making all crates libraries so they can be used by other tools, fixing the weird organization of librustc and its driver, splitting rustllvm out into its own crate with bindings, ditto clang, refactoring the top-level rustc code so that it is more approachable to newcomers.


