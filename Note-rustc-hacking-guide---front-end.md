The compiler front-end includes the lexer, parser and syntax extensions (macros), among other things. The front-end is in a separate crate, called ```syntax```, whose source files live in the ```src/libsyntax``` directory under the main Rust source tree. (The middle and back end are in ```src/rustc```.)

## Syntax extensions
Built-in syntax extensions are located in ```src/librustsyntax/ext```, including the macro-defining construct ```macro```, located in ```simplext.rs```. It also contains ```expand.rs```, which is responsible for traversing and expanding the tree in the first place, and ```base.rs```, which contains helper functions for extensions and registers the extensions for the expander.  

## Parser
Rust's parser, located in ```src/libsyntax/parse/parser.rs``` is a hand-rolled LL-style parser with a small, finite lookahead. Rust's lexer is in ```src/librustsyntax/parse/lexer.rs```.

## Adding a new keyword

If you need to add a new keyword to Rust (a rare occurrence, with hope!) edit one or the other of the functions ```contextual_keyword_table``` or ```restricted_keyword_table``` in ```src/libsyntax/parse/token.rs```. Together, these functions construct the table that parser functions consult when determining which identifiers are keywords.