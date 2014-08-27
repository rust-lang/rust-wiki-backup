The word "Issues" encompassed both bugs and feature requests.

If the meaning is blank, then the meaning is implicit in the name
(contribute a clarifying definition if you believe the meaning is not
self-evident).

If the meaning is a "?", then Felix did not immediately know the
meaning of the label and does not think it is self-evident.

| *TAGS LABELS*            |               Meaning                             |
|:-------------------------|:--------------------------------------------------|
| *AREA TAGS*              |                                                   |
| A-an-interesting-project | A self-contained independent project              |
| A-android                | Related to Android mobile device OS               |
| A-attributes             | Issues with Rust meta-data attributes             |
| A-build                  | Issues bootstrapping rust compiler and runtime    |
| A-codegen                | Issues with rustc code generator                  |
| A-concurrency            | Scheduler, tasks, plus other concurrency idioms   |
| A-debuginfo              | Debug meta-data for integration with `gdb`        |
| A-destructors            | Destructors (static lifetime based; not [finalization][1])  |
| A-diagnostics            | Error reporting (some overlap with A-lint)        |
| A-docs                   | Issues with tutorials, manuals, and API docs      |
| A-driver                 | Issues with wrapper around rustc/rustdoc/etc      |
| A-ffi                    | Foreign Function Interface i.e. interop with C/C++|
| A-freebsd                | Related to FreeBSD operating system               |
| A-frontend               | Issues with rustc frontend (`librustc::front::*`) |
| A-grammar                | Grammar spec (not *implementation*; see A-parser) |
| A-iOS                    | Related to Apple iOS device operating system      |
| A-infrastructure         | Mozilla-hosted services: buildbot, website, etc   |
| A-instrumentation        | Rust library / object code instrumentation        |
| A-libs                   | Rust standard and contributed libraries           |
| A-linkage                | Crate format+tooling for dynamic/static linking   |
| A-lint                   | Warning categorization and tooling in rustc       |
| A-linux                  | Related to Linux operating system                 |
| A-llvm                   | Related to LLVM IR and code-generation library    |
| A-macos                  | Related to Apple Mac OS X operating system        |
| A-metadata               | Crate-embedded data for reflection/compilation    |
| A-parser                 | Parser impl. (not *specification*; see A-grammar) |
| A-pkg                    | Issues with package management e.g. rustpkg       |
| A-plugin                 | compiler plugins and their infrastructure         |
| A-pretty                 | Pretty-printing rust source via `rustc --pretty`  |
| A-regions                | Lifetime semantics; borrow-checker (aka borrowck) |
| A-resolve                | Identifier/name resolution                        |
| A-runtime                | Task scheduler, memory management, core I/O       |
| A-rustdoc                | API documentation extraction and generation tool  |
| A-rusti                  | rusti is a REPL-like dynamic Rust interpreter     |
| A-syntaxext              | syntax extensions aka macros i.e. `macro_rules!`  |
| A-testsuite              |                                                   |
| A-tools                  | For Rust user: syntax highlighting, profiling, etc|
| A-traits                 | Trait system provides bounded polymorphism + OOP  |
| A-typesystem             | Rust's static type system, type check (aka typeck)|
| A-unicode                | Unicode handling, parsing or conversion           |
| A-versioning             | Version numbers/handling for language + libraries |
| A-visibility             | Visibility/privacy: item pub/priv and their access|
| A-windows                | Related to Microsoft Windows operating system     |
| &nbsp;                                                                       |
| *BLOCKER TAGS*           |                                                   |
| B-RFC                    | Blocked on a Request-for-Comment                  |
| B-clarifying             | put label description here                        |
| B-reproduce              | Blocked on a need to reproduce problem locally    |
| &nbsp;                                                                       |
| *EFFORT TAGS*            |                                                   |
| E-easy                   | Effort: easy                                      |
| E-hard                   | Effort: hard                                      |
| E-needtest               | Effort: requires a unit test to be closed         |
| &nbsp;                                                                       |
| *IMPACT TAGS*            |                                                   |
| I-ICE                    | Importance: error internal to rustc               |
| I-cleanup                | Importance: Internal source code cleanup          |
| I-completion             | Importance: fill gaps in existing implementation  |
| I-crash                  | Importance: Rust runtime/program is crashing      |
| I-enhancement            | Importance: Potential tools/language enhancement  |
| I-nominated              | Importance: Nominated for a milestone             |
| I-papercut               | Importance: easily fixable usability bug          |
| I-slow                   | Importance: slow (in compile or running time)     |
| I-wishlist               | Importance: would be nice but can do without      |
| I-wrong                  | Importance: behavior does not match specification |
| &nbsp;                                                                       |
| *PRIORITY TAGS*          |                                                   |
| P-backcompat-lang        | Priority: Language backward-compatible            |
| P-backcompat-lib         | Priority: Library backward-compatible             |
| P-high                   | Priority: High                                    |
| P-low                    | Priority: Low                                     |
| &nbsp;                                                                       |
| *uncategorized tags*     |                                                   |
| metabug                  | Collects several related issues together          |

[1]: http://en.wikipedia.org/wiki/Finalizer
