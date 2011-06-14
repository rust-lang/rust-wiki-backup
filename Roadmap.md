This is a high-level picture of where we're going with Rust over the medium-term. This does not include Mozilla's eventual uses for the language; merely a set of tasks, priorities and rough order that the main developers are mostly agreed on. If you don't see something on this list, or take issue with some of it, please say so!

* [_Complete_] Bootstrap rustc.

* [_Complete_ see [[Compiler Snapshots]]] Implement a stable incompatible-feature-migration strategy for rustc.

* [_In Progress_] Perform the long-delayed set of "agreeable" syntax changes (mostly just tweaking keywords and literals). We've been deferring most discussion of syntax for a while, but have built up (casually, mostly off the record in private notes) a substantial set of less-contentious changes that we're expecting to discuss, find consensus on, and implement as a batch. This will involve bulk-editing all existing rust code in the repository (rustc, standard library and tests).

* [_In Progress_] Complete parts of rustc not necessary for bootstrapping but represented in the testsuite. This includes the typestate system, the tasking system, the syntactic extension system, the unwinder and the garbage collector.

* [_In Progress_] Introduce uniquely-owned boxes and whatever semantic modifications are required to handle this addition to the memory model. Unique ownership permits several important optimizations, such as elimination of reference counting overheads and multi-threaded read access to 'pinned' data structures.

* Extend the runtime library. At present the runtime services are quite minimal and incomplete: thread domains are only partly functioning, process domains do not exist at all, and the logging service some of the required filtering mechanisms. Some runtime services can probably be rewritten in rust while working on this.

* [_In Progress_ see [[Self Types]] and [[Object Types]]] Introduce the keyword 'self' for supporting self-types and self-dispatch in the object system as well as static "method"-like (value.function()) dispatch for non-object types. Possibly consider a looser static dispatch scheme similar to haskell typeclasses for the latter.

* [_In Progress_] Possibly introduce a form of environment-capture or anonymous function literal, if we can agree on that. This is unclear but is a regular feature-request; many people seem to dislike the current 'bind'-based closure system. We've got some design work to do here to see if we can produce something better.

* [_In Progress_] Develop components of a library ecosystem. This means getting the crate versioning and metadata system pinned down to a satisfactory degree, as well as putting online a registry of 3rd party installable crates and some network-enabled librarian logic, either in rustc or a separate tool.
