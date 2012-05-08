The compiler front-end includes the lexer, parser and syntax extensions (macros), among other things. The front-end is in a separate crate, called ```syntax```, whose source files live in the ```src/librustsyntax``` directory under the main Rust source tree. (The middle and back end are in ```src/rustc```.)

## Adding a new keyword

If you need to add a new keyword to Rust (a rare occurrence, with hope!) edit one or the other of the functions ```contextual_keyword_table``` or ```restricted_keyword_table``` in ```src/librustsyntax/parse/token.rs```. Together, these functions construct the table that parser functions consult when determining which identifiers are keywords.