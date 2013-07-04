rustdoc currently needs some love, and I (cmr) am going to give it some.

# Architecture

- take AST, make pretty docs. easy!

# Pain points with current rustdoc

- `Implementation of Foo for Bar where <useless stuff>` (from SL. where -> given?) 

# New features

- Unformatted JSON/XML (undecided) output
- More complete TOCs, indexes, links to all items documented
- Interactive browser interface (live search, toggle platform cfg's)
- every symbol should be able to be linked to its documentation (eg, types, methods, functions)
- new attribute that instructs rustdoc to inline a nested module into its parent's documentation
- document macros
- navigate visibility graph and don't generate docs for stuff which isn't visible
- document macro-generated code
- link to source
- "a cross-reference with a list of all traits declared in all modules would be very useful. Developers, particularly people writing libraries, would have a big "to-do" list of all the traits their new types could implement."
- your feature requests here!

# New bugs

- your bug requests here!

# Progress

- this will track pull requests and status updates as I create them.