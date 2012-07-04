This page used to cover early (pre-0.1) tasks on the development roadmap for Rust. It is now an outline of where we plan to take the compiler over the course of the next several releases, ending in a 1.0 release that we commit to supporting over an extended time period. Where possible, we will link to bugs in the issue tracker.

Items on this page will move to the [[Doc detailed release notes]] page as they are completed.

## Miscellaneous cleanups

### Change closure-kind syntax

Closures have to encode their kind (whether they copy their environment, uniquely copy the unique parts, or only hold a safe reference to it). Currently this is indicated by a sigil: `fn@` or `fn~` or `fn&` or such. We're likely to change this to one of the kind names trailing the word `fn`. That is, call it `fn:copy` or `fn:send`.

## Compilation and linkage-model changes

There are a bunch of changes in here, all inter-related. They're mostly agreed-on though.

### Change `crust` and `native` to `extern`

This just gets rid of a couple keywords and feeds into the third item. When an item is marked with `extern`, if it does not have a body it is a declaration of code-written-in-C (or some other foreign language) linked to rust; if it has a body it is a declaration of code-written-in-rust that should be exposed via some foreign ABI.

### Separate form of module import: `mod foo = bar;`

This has to do with making the resolve pass coherent. In the old resolve code, resolving modules and resolving items glob-imported _from_ modules was intermixed, and could lead to incoherence of the algorithm. In the new resolve code (landing in 0.3) there is a separation of passes: module-imports are not run through globs, only module-to-module renamings, and are resolved _first_, and then all imports through modules (including glob-imports) are resolved _after_. The module-import syntax changes to `mod foo = bar;` to reflect this change.

### Change `use` to mean `import`, remove `import`, make crate-linkage use `extern mod foo = (...);`

Many Rust programmers stub their toes on the difference between `import` and `use`; both "read like" they should somehow make-available the elements in the target module. Since we are getting rid of `export` (see below) there is an asymmetry in the keywords anyways, so we remove the keyword `import`, switch `use` to mean what `import` currently means, and denote crate-linkage through `extern mod foo = (...)`.

### Remove the distinction between crate files and source files

Crate files are at this point mostly an artefact of earlier beliefs that turned out not to be true (or convenient) in the compilation model. We believe they are now doing more harm than good, and their features can be presented as an "early pass" in the compilation model of a single tree of source files. Inter-file linkages will be given (within a crate) by the form `mod foo = "path.rs"`.

### Generalized variables and conditionals in the attribute system.

The attribute system has served us well so far for conditional compilation but at times we find it not quite powerful enough. In particular, the ability to bind variables declaratively, as well as conditionally evaluate _any_ attribute, seems lacking. We'll expand the attribute system to handle these cases, possibly changing its syntax slightly along the way.

### Language-level versioning, shebang comments.

For long-term source compatibility, we wish to make it possible (and in the case of packaged software for distribution via cargo, possibly mandatory) to tag rust source files with the language version they are written against. We'll be doing this outside the main lexical grammar, in a very simplified "pre-parse" grammar that we can expect to remain stable indefinitely. This sub-grammar is only enough to express version tags (though it uses a `#`-comment form that happens to overlap with the needs of `#!` at the first line of a file for running as a script).

### Version attribute, API versioning

Currently a crate has a single version, which is mangled into all the symbols in the crate as well as the crate filename. This is not quite correct. What we want is a per-item version attribute (with a per-crate _default_) that is mangled into each symbol, but _not_ the output filename, such that the compiler can tolerate compiling multiple versions of the same API inside a single output file. This should be mostly invisible to users.