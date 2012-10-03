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

