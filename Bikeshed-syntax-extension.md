## `#macro`-style

The expander allows for simple macro definitions. Sadly, we haven't had time to make them hygienic or look at macro-defining macros.

`#macro[⋯]` is the macro that defines a new macro in a Macro By Example style.

There also exist some predefined syntax extensions, such as `#fmt` and `#concat_idents`.

## Procedural macros

Procedural macros are more complicated; we haven't studied them much yet. One possible approach is to embed a language that's not Rust for procedurally defining macros. This brings its own set of issues, but simplifies other matters.

# Issues
## Macro definition interface
A macro is at least a function from syntax to syntax. It may need an error reporting facility, and possibly a way to communicate to the parser to deal with the various nonterminals it may expect.

## Nonterminals: expr/type/item/view_item
(this discussion is somewhat obsolete)

Macros will need to be able to take various different kinds of nonterminals. 

* One (terrible) option is for macros to take blocks, and will specify a syntactically valid place to put, say, a type in the block; the macro will then throw out everything other than the type in that position.
* We could provide syntax that marks which nonterminal is coming up (a sort of sigil-like syntax).
* We could leave off sigils, and write a parser that chooses one possibility over the others deterministically (and make sure that the syntax allows us to disambiguate). 
* We could have macros tell the parser what nonterminals they expect. This will require some form of dependent parsing, and we're not sure how it interacts with macro-defining macros. pauls believes that it will work fine, but no one has done it before.

# Invocation syntax
## Current
    Expr → "#" Path Expr
    Expr → "#<" Type ">" | "#{" Item "}"

At the moment, macros can only destructure `#<>`, `#{}`, and `[]`. It suffices, but we might like to be able to pun on existing expression syntax. 

## Proposed
    Expr → "#" Path BalancedLexemes
    BalancedLexemes → "(" BalancedLexemes * ")"
    BalancedLexemes → "[" BalancedLexemes * "]"
    BalancedLexemes → "{" BalancedLexemes * "}"
    BalancedLexemes → AnyOtherLexeme

The macro system will need a way to call back into the parser to turn the lexemes into actual syntax. This is non-trivial, especially ergonomics-wise, but should be doable, and would allow for a great deal of flexibility in macro definition.

## Expansion order
Expansion should be outside-in. This means that macros will need to expand code that has macro invocations in it. This means that the AST will need nodes representing macro invocations.

# Benchmarks (brainstorm)

* RPC-enabled object type generator
* Parser generator
* `lambda-match`
* `and`
* `when`/`unless`
* declarative data struture
* fixed-length arrays
* `cut`, from [SRFI 26](http://srfi.schemers.org/srfi-26/srfi-26.html).

# Super-simple syntax extension structure

    #simpleext(m1,param1,param2,param3,...,paramn,some_expr)
Then, `m1` is defined as an extension taking `n` parameters, named `param1`-`paramn`. Those identifiers may appear free in various expression positions in `some_expr`; the argument syntax will be substituted at that point.

# Ideas concerning attributes

It may be desirable to access or even generate attributes via macros. Though that complicates matters, it also could be really powerful as scoped metadata (i.e. attributes) could be used to control macro expansion.
