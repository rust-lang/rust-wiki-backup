We currently have two stories with regards to macros. The reader should be aware that they don't even contradict each other: it's possible to build a system that naturally follows both stories.

## `#macro`-style

The expander allows syntax extensions to add things to the syntax extension table.

Let `#macro(macro_name, ...)` be the macro that adds `macro_name` as a new macro. Presumably, the definition takes place in some pattern-matching language. It would be possible to write new syntax extensions `#my_macro` that use other languages.

(If rustc provides the ability to run Rust programs as a library, then it might even be possible to write `#rust_macro`, which would take a macro definition in Rust as an argument.)

The presence of these "second-generation" macros means that requiring "first-generation" macros to reside in a separate crate is not as onerous a burden, since same-file macro definitions are also possible.

## single-phase-file-style

If one file (module?) `import_for_syntax`es another file (module?), the exported functions are brought in as syntax extensions.  Each file (module?) will only contain things in the same phase (to use a term from _You Want it When?_). We'll provide an API for syntax extensions, containing, at the least, some sort of syntax tree-manipulating functions. Ideally, syntax-quotation will also be present.

After that is complete, consider the following tasks:

* implement a term-rewriting (match/splice)-style macro system, possibly allowing macros to be used in the same file they are defined in.
* possibly allow macros that know the `typeof` expressions, or macros that preserve types. (Both of these are very hard. For the former, dherman believe that a simple subset may be possible. No one is very interested in the latter, but pauls things that they're mutually exclusive, just as a warning.)