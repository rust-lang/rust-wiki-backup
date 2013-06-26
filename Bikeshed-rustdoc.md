rustdoc currently needs some love, and I'm going to give it some.

# Architecture

- a backend that does all the heavy lifting: extract type signatures, doc strings, etc. this is basically every current pass until `sort_item_name_pass`. This backend would return some sort of document (as in json/xml) with a list of: traits, functions, typedefs, constants, uses, mods, extern mods, enums, and toplevel attributes. This would be on a per-file basis. Everything would include a reference to the file and line number it was defined. This allows linking to the source like Python does in the docs sometimes.
- filters which take these documents, which has pretty much everything in a nice stringified form and does further processing. These would create separate indexes and ToCs, merge documents where useful and so forth. These output a document of the same format as the backend with some (optional) ancillary data for a frontend to maybe use. These filters would do things like remove doc(hidden), extract brief descriptions, remove extra indentation, etc.
- various frontends which take these documents and output something useful (markdown for pandoc, rst for sphinx, what have you).

This is much more flexible and should be more parallelizable than current rustdoc (which seems to be rather serial, but I haven't done a thorough review of it yet). It also has the benefit of being able to serve as a backend for more than just generating static docs with pandoc, specifically being able to output a format usable in an IDE for inline-documentation (much like java and .net have).

Diagram:

```
                  +--------+   +-------+  +--------+
                  |foo.rs  |   |bar.rs |  |baz.rs  |
                  +---+----+   +---+---+  +----+---+
                      |            |           |
                +-----|----rustdoc-|-----------|-----+
                      |            |           |
                      v            v           v
                  +--------+   +-------+  +--------+
                  |extract |   |extract|  |extract |
                  +---+----+   +---+---+  +----+---+
                      |            |           |
                      |            |           |
                      v            v           v
                  +--------+
                  |filter 1|    ....
                  +---+----+       +           +
                      |            |           |
                      |            |           |
                      v            |           |
                  +--------+       |           |
                  |filter n|       |           |
                  +--------+       |           |
                      +            |           |
                      |            |           |
                      |            |           |
                      +            +           +
                      |            |           |
                      |            |           |
                      |            v           |
                      +------>+----------+<----+
                              |generation|
                              +----------+
```

At any point in the above, rustdoc can stop and spew out JSON/XML.

# Pain points with current rustdoc

- `Implementation of Foo for Bar where <useless stuff>` (from SL. where -> given?) 

# New features

- Unformatted JSON/XML (undecided) output
- More complete TOCs, indexes, links to all items documented
- Interactive browser interface (live search, toggle platform cfg's)
- every symbol should be able to be linked to its documentation (eg, types, methods, functions)
- new attribute that instructs rustdoc to inline a nested module into its parent's documentation
- your feature requests here!

# New bugs

- your bug requests here!

# Progress

- this will track pull requests and status updates as I create them.