Attending: brson, nmatsakis, graydon

Topics:

- Closures: unique and otherwise
  - Have to make closed over state immutable and immobile (treat as reference)
  - Merge bare functions (fn) and unique closures so that we have:
    - functions: sendable
    - lambdas: shared closures, not sendable
    - blocks: only within current stack frame
  - C interopability? Unclear how this will be managed

- Performance on x86_64
  - Culprit seems to be poor hashtable lookups
  - Maybe hashtables are degrading into linked lists?

- Object system
  - Insufficient quorum to really discuss

- Status of stack growth
  - Working on mac/linux but for unwinding
  - Need to impl. on windows
  - Unwinding unsupported on windows because LLVM doesn't support it
  - We might want to move away from DWARF and keep our own shadow stack

- Blockers for 0.1 release
  - Mostly small stuff
  - Graydon added some version of cargo to the list
    - "first thing we'll want to do is package up libraries"
