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

# Issues
## Macro definition interface
A macro is at least a function from syntax to syntax. It may need an error reporting facility, and possibly a way to communicate to the parser to deal with the various nonterminals it may expect. It's possible that macros will transform syntax directly into new macros, to be fed back to the expander. That's a weird thing to do, though.

## Nonterminals: expr/type/item/view_item
Macros will need to be able to take various different kinds of nonterminals. 

* One (terrible) option is for macros to take blocks, and will specify a syntactically valid place to put, say, a type in the block; the macro will then throw out everything other than the type in that position.
* We could provide syntax that marks which nonterminal is coming up (a sort of sigil-like syntax).
* We could leave off sigils, and write a parser that chooses one possibility over the others deterministically (and make sure that the syntax allows us to disambiguate). 
* We could have macros tell the parser what nonterminals they expect. This will require some form of dependent parsing, and we're not sure how it interacts with macro-defining macros. pauls believes that it will work fine, but no one has done it before.

## Syntax
We've provisionally used `#` as a sigil to indicate a macro expansion. But it would be nice to have more flexibility than a flat list of syntactic arguments, so some other kind of delimiter (or possibly a way to specify keywords) may be necessary.

## Expansion order
Expansion should be outside-in. This means that macros will need to expand code that has macro invocations in it. This means that the AST will need nodes representing macro invocations.

# Benchmarks (brainstorm)

* RPC-enabled object type generator
* Parser generator
* `lambda-match`
* `and`
* `when`/`unless`
* declarative data struture

# Super-simple syntax extension structure

    #simpleext(m1,param1,param2,param3,...,paramn,some_expr)
Then, `m1` is defined as an extension taking `n` parameters, named `param1`-`paramn`. Those identifiers may appear free in various expression positions in `some_expr`; the argument syntax will be substituted at that point.