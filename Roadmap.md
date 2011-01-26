This is a high-level picture of where we're going with Rust over the medium-term. This does not include Mozilla's eventual uses for the language; merely a set of tasks, priorities and rough order that the main developers are mostly agreed on. If you don't see something on this list, or take issue with some of it, please say so!

1. Bootstrap rustc. The current state of affairs is that rustboot is buggy, incomplete, and generates horrible code. Rustc on the other hand generates very good code and is much less buggy, but is even less complete. Rustboot can build rustc, but rustc cannot build itself. So we are stuck having to rely on rustboot. We are presently focusing all energy on bootstrapping rustc -- that is, completing enough feature work that it can compile itself -- so that we can discard rustboot. This blocks all other points on the roadmap. Later roadmap entries may occur in parallel with one another, or in a different order than is presented here. But #1 here is definitely #1.

2. Implement a stable incompatible-feature-migration strategy for rustc. This will require some careful thought about the logic of self-modification, and some automation for archiving executable snapshots of rustc when we introduce changes that are not backwards compatible. Adjust the makefile to support fetching the development basis (rustc, standard library and runtime) in binary form from an archival server, while developing. While we're in there, tidy up the build logic: strip out all support for rustboot once it's discarded, split the configuration stage off to support out-of-tree builds, etc.

3. Perform the long-delayed set of "agreeable" syntax changes (mostly just tweaking keywords and literals). We've been deferring most discussion of syntax for a while, but have built up (casually, mostly off the record in private notes) a substantial set of less-contentious changes that we're expecting to discuss, find consensus on, and implement as a batch. This will involve bulk-editing all existing rust code in the repository (rustc, standard library and tests). Might also involve a full implementation of a pretty-printer in rustc.

4. Complete parts of rustc not necessary for bootstrapping but represented in the testsuite. This includes the typestate system, the tasking system, the syntactic extension system, the unwinder and the garbage collector.

5. Introduce uniquely-owned boxes and whatever semantic modifications are required to handle this addition to the memory model. Unique ownership permits several important optimizations, such as elimination of reference counting overheads and multi-threaded read access to 'pinned' data structures.

6. Extend the runtime library. At present the runtime services are quite minimal and incomplete: thread domains are only partly functioning, process domains do not exist at all, and the logging service lacks any of the required filtering mechanisms. Some runtime services can probably be rewritten in rust while working on this.

7. Introduce the keyword 'self' for supporting self-types and self-dispatch in the object system as well as static "method"-like (value.function()) dispatch for non-object types.

8. Possibly introduce a form of environment-capture or anonymous function literal, if we can agree on that. This is unclear but is a regular feature-request; many people seem to dislike the current 'bind'-based closure system. We've got some design work to do here to see if we can produce something better.

9. Develop components of a library ecosystem. This means getting the crate versioning and metadata system pinned down to a satisfactory degree, as well as putting online a registry of 3rd party installable crates and some network-enabled librarian logic, either in rustc or a separate tool.
