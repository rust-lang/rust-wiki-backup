(draft)

After bootstrapping, we'll have a weird, dependant relationship to our own compiler binaries. Someone starting a checkout will have to get one from a central place, as will people who pull in changes from others.

In today's (April 8 2011) meeting, we decided:

- We'll test using a three-stage (1-3) test process, with the first stage being built by a snapshot compiler (stage0) and the third stage being expected to be the same as the second (either by LLVM code or by code/data sections) -- i.e. a 'fixpoint' where recompiles don't introduce new changes.

- There will be a file (snapshot registry) in the repository that tracks the URLs of compilers that can be used to build the different revisions (by git hash).

- There will be an automated process to kick off some servers (different platforms) to build a snapshot compiler, upload it, adjust the registry file to contain the actual URL and hash of the binary, and push to the central repo.

- The tracking file marks both snapshots and intermediary commits that have to be built to move from one snapshot to another. For example, say we change the syntax:

  - We start with a compiler (from git and snapshot) that is written in the old syntax and compiles the old syntax.
  - We modify the code to compile the new syntax, still using the old in the compiler code. This one can compile itself, but won't fixpoint, so it's not a checkpoint
  - We now modify the syntax of the compiler code, and compile it with the in-between compiler. This is again a fixpoint.

- Thus, to be able to 'replay' the process (coming from the bootstrap compiler, going through our log of git commits and snapshots to the latest compiler), we must also mark the 'transition' commits that don't have snapshots, but have to be compiled to get from the previous snapshot to the next.

- The makefile will be smart about making sure the correct snapshot is downloaded to your stage0 directory. If your local snapshot registry contains snapshots that don't have a URL yet, it'll leave the stage0 alone. If the top snapshot does have a url and digest, it'll compare the listed digest with the file present, and update if necessary.

- When working on an incompatible change, you'll at some point want to use your transitional (see above) compiler instead of a stage0 snapshot. The makefile can detect that the top entry in the snapshot registry is transitional (there will be notation for this), and it will use a compiler in another directory, say stage0.5, that you built before as part of the transition process.