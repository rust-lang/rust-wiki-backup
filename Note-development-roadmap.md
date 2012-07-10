This page used to cover early (pre-0.1) tasks on the development roadmap for Rust. It is now an outline of where we plan to take the compiler over the course of the next several releases, ending in a 1.0 release that we commit to supporting over an extended time period. Where possible, we will link to bugs in the issue tracker.

Please note: when we do stabilize for a 1.0 release, it will _not_ mean we are freezing the language for all time; it means we'll be branching and supporting a stable branch and a versioning strategy that "works" for an extended period (i.e. "years"), as well as committing to providing tools to help migrate code forward (_help_, not necessarily automate), when and if breaking changes are made in future versions.

Items on this page will move to the [[Doc detailed release notes]] page as they are completed.

## Miscellaneous cleanups, mostly syntax

### Module-separator change (possible)

We may change the module-separator from `::` back to `.`. There's little consensus on this at the moment.

### Labeled loops

([#2216](https://github.com/mozilla/rust/issues/2216)) Loops currently cannot carry labels, which makes breaking from deep within a loop difficult. There's a pretty clear way to implement this, it just requires some care to avoid clashing with nearby syntax.

### Macro-invocation change, general macro-system rewrite

([#2387](https://github.com/mozilla/rust/issues/2387)) Macros are changing to work on uniform balanced token-trees -- much like s-expressions -- rather than the existing system which is based on expressions (with separate quoters for type, item and pattern grammars). Concurrently, we are likely to introduce the "new" macro system with a new syntax that is a bit easier on the eyes: `macro_name! args`, where args is a balanced token-tree. There's still some disagreement on whether this reads better than `#macro_name(...)` but we'll decide at some point and stick with one or the other.

### Raw-strings rather than balanced-character custom lexemes

([#2755](https://github.com/mozilla/rust/issues/2755)) This is a minor change that should effect no code presently; we'll be introducing a "raw string" form (possibly with a variety of legal delimiters) that does _not_ balance the delimiter characters, so requires internal escaping of only the delimiter. This will replace the proposed (but never implemented) character-balanced custom lexeme syntax in the syntax-extension system. The cost of having a non-regular token grammar was deemed not worth the benefit, and most of the use cases for the latter are easily handled by the former.

### Change closure-kind syntax

Closures have to encode their kind (whether they copy their environment, uniquely copy the unique parts, or only hold a safe reference to it). Currently this is indicated by a sigil: `fn@` or `fn~` or `fn&` or such. We're likely to change this to one of the kind names trailing the word `fn`. That is, call it `fn:copy` or `fn:send`.

### `alt` syntax changes

The `alt` form requires braces, which is too heavyweight for some use cases. There is mostly consensus on moving to `=>` to separate patterns from expressions and dropping the braces. There is also the possibility (no consensus) that the name `alt` will change to `match`, to better match other languages.

### Semicolons and return values

There is the possibility that return values will need to be explicitly ignored if the programmer desires this; with this change we can make trailing semicolons optional, eliminating a frequent source of toe-stubbing.

### Floating point literals

There is general consensus that floating point literals are too long; `1.0f` will likely become sugar for `1.0f32`. This is purely a backwards-compatible change.

### Capitalization

Some Rust projects are beginning to capitalize types. We will need to come to a consensus on the house style.

## OO-system changes

### Extend interfaces to full traits

([#2794](https://github.com/mozilla/rust/issues/2794)) Traits are interfaces that carry method implementations and requirements on the self-type; they can be used to achieve code reuse without introducing much of the pain that comes from conventional inheritance: they can be mixed in any order, can be declared independent from the type they affect, and are compiled independently for each type that implements them (indeed, will inline and specialize exactly as any other method will). This work will replace the `iface` keyword with `trait` and provide (hopefully) most of what people miss when writing rust in OO style.

### Minimize classes

([pcwalton:max-min-classes](http://pcwalton.github.com/blog/2012/06/03/maximally-minimal-classes-for-rust/)) Along with introducing traits for code-reuse, we will likely trim down the `class` construct in the current version of rust to carry only the minimum necessary to complement traits: a nominal record type with some form of access control on its fields. This may subsume some or all of the space currently occupied by `class`, record types or `enums`; discussion here is still ongoing.

### Enforce implementation coherence

([pcwalton:impl-coherence](http://pcwalton.github.com/blog/2012/05/28/coherence/)) Currently when a user calls a method defined by an `impl`, the code selected is chosen based on the `impl`s that are imported into the _client code_'s scope. This was chosen as a way to be unambiguous about selecting implementations -- a problem in any typeclass system -- but in practice it has been very confusing for users: many are unable to tell why a method can or cannot be seen due to the presence or absence of imports. It also leaves open a few "gotchas" to do with passing data values between modules with different visible imports, apparently calling the same methods or instantiating the same `iface`s, but selecting different `impl`s.

One way or another (there are at least 2, maybe 3 ways in discussion) we will be enforcing that only one implementation of an interface (or trait) exists per type, and removing the relationship between `impl` selection and imported symbols altogether. This may happen by a per-crate static check during compile time, or it may happen by construction (making it impossible to declare implementations outside traits).

### Destructors change to an iface

Currently we have a full "language level" construct for a value-with-a-destructor: a `drop` block inside a class. This is still more machinery than strictly necessary, and we wish to transition to merely interpreting the presence of a distinguished interface (`intrinsic::drop`) as indicative of a value having a destructor. All the same kind-related rules will apply, this is just a matter of removing surface machinery from the language.

## Memory model changes

### Remove argument modes and last-use analysis, use unary `move` and borrowed (region) pointers

([#2030](https://github.com/mozilla/rust/issues/2030)) This is simply removing code that has proven too difficult to predict and control in practice, and is now redundant with full monomorphization and first class borrowed pointers.

### Turn `mut` into a full type constructor

([nmatsakis:mut](http://smallcultfollowing.com/babysteps/blog/2012/05/31/mutability/)) We have tried this several times in the past to little success, but Niko believes there is a reasonably good chance that `mut` will work better as a full type constructor rather than a slot-qualifier. This is one of the larger unknowns in the current roadmap.

### Add a `const` kind and freeze/thaw operations

Rust types are stratified into different "kinds": the `copy` kind or the `send` kind, for example. These appear in the type system as special interfaces but their existence and the conformance of a type to the interface is implemented by the compiler, by analyzing the structure of the type. For example, the compiler will not make a type with a destructor, or a function that captures its environment by reference, a member of the `copy` kind, as it cannot be sensibly copied; likewise a type with `@`-values in its substructure cannot be safely sent between tasks, so will not be put in the `send` kind.

A new kind is appearing in 0.3, called `const`, that applies to `send`-kind types that additionally have no `mut` fields anywhere in their substructure. These types are, for the sake of concurrent access, _read only_ values that can be safely read by multiple threads at the same time. They're as good as values held in the read-only segment of a program.

The kind exists to support various library functions, for example a reference-counted multi-reader variable (called `core::arc`) as well as, in the future, a function `freeze` that can be used to convert a `send` value to a `const` value by casting it to a type with all `mut` type modifiers removed. We expect this to be useful when implementing publish/subscribe patterns between concurrent tasks.

### Type-reflection system

([#1498](https://github.com/mozilla/rust/issues/1498)) A very preliminary form of will arrive in 0.3 and be extended over time to cover many tasks: type descriptors contain a compiler-generated function that calls visitor-methods on a predefined intrinsic visitor interface. This enables reflecting on a value without knowing its type (with some supporting library work). Much existing code will gradually shift over to this interface, as it subsumes a number of other tasks the compiler and runtime are currently doing as special cases.

## Compilation and linkage-model changes

There are a bunch of changes in here, all inter-related. They're mostly agreed-on though.

### Separate form of module import: `mod foo = bar;`

(also [#2082](https://github.com/mozilla/rust/issues/2082)) This has to do with making the resolve pass coherent. In the old resolve code, resolving modules and resolving items glob-imported _from_ modules was intermixed, and could lead to incoherence of the algorithm. In the new resolve code (landing in 0.3) there is a separation of passes: module-imports are not run through globs, only module-to-module renamings, and are resolved _first_, and then all imports through modules (including glob-imports) are resolved _after_. The module-import syntax changes to `mod foo = bar;` to reflect this change.

### Change `export` to `pub`, move it from top-of-module to individual items.

([#2300](https://github.com/mozilla/rust/issues/2300)) There's some tension in readability between "ease of scanning a module's exports" and "ease of reading the code and knowing which item is exported when you're looking at it." Ultimately we came down on the side of maintainability: that it's less work for a maintainer to mark the items where they occur, rather than scrolling back and forth between export-list and item definitions. Along with moving exports to the items themselves, we'll be changing the terms to use the same keywords used for access control in classes: `pub` and `priv`.

### Change `use` to mean `import`, remove `import`, make crate-linkage use `extern mod foo = (...);`

(also [#2082](https://github.com/mozilla/rust/issues/2082)) Many Rust programmers stub their toes on the difference between `import` and `use`; both "read like" they should somehow make-available the elements in the target module. Since we are getting rid of `export` (see below) there is an asymmetry in the keywords anyways, so we remove the keyword `import`, switch `use` to mean what `import` currently means, and denote crate-linkage through `extern mod foo = (...)`.

### Remove the distinction between crate files and source files

([#2176](https://github.com/mozilla/rust/issues/2176)) Crate files are at this point mostly an artefact of earlier beliefs that turned out not to be true (or convenient) in the compilation model. We believe they are now doing more harm than good, and their features can be presented as an "early pass" in the compilation model of a single tree of source files. Inter-file linkages will be given (within a crate) by the form `mod foo = "path.rs"`.

### Generalized variables and conditionals in the attribute system.

([#1242](https://github.com/mozilla/rust/issues/1242) and [#2119](https://github.com/mozilla/rust/issues/2119)) The attribute system has served us well so far for conditional compilation but at times we find it not quite powerful enough. In particular, the ability to bind variables declaratively, as well as conditionally evaluate _any_ attribute, seems lacking. We'll expand the attribute system to handle these cases, possibly changing its syntax slightly along the way.

### Language-level versioning, shebang comments.

([#2159](https://github.com/mozilla/rust/issues/2159) and [#1772](https://github.com/mozilla/rust/issues/1772)) For long-term source compatibility, we wish to make it possible (and in the case of packaged software for distribution via cargo, possibly mandatory) to tag rust source files with the language version they are written against. We'll be doing this outside the main lexical grammar, in a very simplified "pre-parse" grammar that we can expect to remain stable indefinitely. This sub-grammar is only enough to express version tags (though it uses a `#`-comment form that happens to overlap with the needs of `#!` at the first line of a file for running as a script).

### Version attribute, API versioning

([#2166](https://github.com/mozilla/rust/issues/2166)) Currently a crate has a single version, which is mangled into all the symbols in the crate as well as the crate filename. This is not quite correct. What we want is a per-item version attribute (with a per-crate _default_) that is mangled into each symbol, but _not_ the output filename, such that the compiler can tolerate compiling multiple versions of the same API inside a single output file. This should be mostly invisible to users.

## Library work

### Condition-handler system

0.3 will introduce an API for setting and retrieving task-local data. We'll build on top of this to provide dynamic-scoped variables (keyed by global constants), on top of that, condition-handlers that can be used to recover from errors at the site of the error, or else fail. This should hopefully address many of the remaining use-cases people have in mind for catchable exceptions.

### IO library update

The `core::io` library is due for a careful refactoring in terms of traits, condition handlers, and similar "new" abstractions that the language and libraries support. While this is true of the entire core and standard libraries, `io` is particularly important in our work due to its pervasive use and numerous implementation variants.

### New serialization backends

Once `core::io` is refactored, the `std::serialization` library will grow several more implementations, and once we decide on a preferred backend we'll migrate the compiler metadata tables to use it. This should be reasonably unnoticed by users, but will break binary compatibility between versions when we make the change.

### Task management

The task creation interface will be enhanced, adding more flexibility for grouping tasks according to which should die when a single task fails. There will be options to spawn tasks with bidirectionally-linked failure propagation (i.e., all tasks in a group dies if one dies), no failure propagation (failure status can be sent via ports), and perhaps also unidirectional failure propagation (a failing parent task takes its children with it).