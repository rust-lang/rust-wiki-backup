This page used to cover early (pre-0.1) tasks on the development roadmap for Rust. It is now an outline of where we plan to take the compiler over the course of the next several releases, ending in a 1.0 release that we commit to supporting over an extended time period. Where possible, we will link to bugs in the issue tracker.

Please note: when we do stabilize for a 1.0 release, it will _not_ mean we are freezing the language for all time; it means we'll be branching and supporting a stable branch and a versioning strategy that "works" for an extended period (i.e. "years"), as well as committing to providing tools to help migrate code forward (_help_, not necessarily automate), when and if breaking changes are made in future versions.

Items on this page will move to the [[Doc detailed release notes]] page as they are completed.

## Miscellaneous cleanups, mostly syntax

### Labeled loops

([#2216](https://github.com/mozilla/rust/issues/2216)) Loops currently cannot carry labels, which makes breaking from deep within a loop difficult. There's a pretty clear way to implement this, it just requires some care to avoid clashing with nearby syntax.

### Raw-strings rather than balanced-character custom lexemes

([#2755](https://github.com/mozilla/rust/issues/2755)) This is a minor change that should effect no code presently; we'll be introducing a "raw string" form (possibly with a variety of legal delimiters) that does _not_ balance the delimiter characters, so requires internal escaping of only the delimiter. This will replace the proposed (but never implemented) character-balanced custom lexeme syntax in the syntax-extension system. The cost of having a non-regular token grammar was deemed not worth the benefit, and most of the use cases for the latter are easily handled by the former.

### Change closure-kind syntax

([#3056](https://github.com/mozilla/rust/issues/3056)) Closures have to encode their kind (whether they copy their environment, uniquely copy the unique parts, or only hold a safe reference to it). Currently this is indicated by a sigil: `fn@` or `fn~` or `fn&` or such. We're likely to change this to one of the kind names trailing the word `fn`. That is, call it `fn:Copy` or `fn:Send`.

### Floating point literals

([#3059](https://github.com/mozilla/rust/issues/3059)) There is general consensus that floating point literals are too long; `1.0f` will likely become sugar for `1.0f32`. This is purely a backwards-compatible change. Alternately, we may extend integer literal inference to floats ([#3059](https://github.com/mozilla/rust/issues/3059))

## OO-system changes

### Destructors change to a trait

([#3061](https://github.com/mozilla/rust/issues/3061)) Currently we have a full "language level" construct for a value-with-a-destructor: a `drop` block inside a struct. This is still more machinery than strictly necessary, and we wish to transition to merely interpreting the presence of a distinguished interface (`intrinsic::drop`) as indicative of a value having a destructor. All the same kind-related rules will apply, this is just a matter of removing surface machinery from the language.

## Memory model changes

### Eliminating the significance of last-use analysis, use of unary `move` instead

([#2033](https://github.com/mozilla/rust/issues/2633)) At the moment last-use analysis still drives injection of a substantial number of implicit unary `move` operators. This has proven too counter-intuitive to remain; at most we expect last-use analysis to drive a warning about large implicit copies, but nothing more.

## Compilation and linkage-model changes

### Remove the distinction between crate files and source files

([#2176](https://github.com/mozilla/rust/issues/2176)) Crate files are at this point mostly an artifact of earlier beliefs that turned out not to be true (or convenient) in the compilation model. We believe they are now doing more harm than good, and their features can be presented as an "early pass" in the compilation model of a single tree of source files. Inter-file linkages will be given (within a crate) by the form `mod foo = "path.rs"`.

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

