A rustdoc is just a snippet of markdown that is associated in some way
with a unit of Rust code. There is very little structure imposed on the
documentation itself. Unlike javadoc and other tools there is no internal
markup for identifying things like arguments. How the documentation is
written is a matter of convention.

Rustdocs can be as simple as `Returns a slice` or they can be more involved:

    Adopt a slice of a string as a node.

    If the slice is longer than `max_leaf_char_len`, it is logically split
    between as many leaves as necessary. Regardless, the string itself
    is not copied

    # Arguments

    * byte_start - The byte offset where the slice of `str` starts.
    * byte_len   - The number of bytes from `str` to use.

    # Safety note

    Behavior is undefined if `byte_start` or `byte_len` do not represent
    valid positions in `str`

    # Example

        let slice = of_substr(s, 0u, 1u);

### Where the rustdocs live

Rustdocs must be put inside `doc` attributes, which are name-value
attributes called "doc" with a string value.

    #[doc = "Engage thrusters"]
    fn thrusters_on() { ... }

To write a more involved rustdoc you just use a multiline string:

    #[doc = "
    Engage thrusters

    Begin heating toast at the temperature specified by the temperature dial
    "]
    fn thrusters_on() { ... }

Because rustdocs can go anywhere attributes can go it is also possible
to write function documentation inside the function definition by
terminating the `doc` attribute with a semicolon.

    fn thrusters_on() {
        #[doc = "Engage thrusters"];
        ...
    }

Putting rustdocs inside strings works fairly well most of the time,
but does result in some quirks. In particular you must remember to
escape double quotes. Because of this limitation I usually end up
using single quotes for quoted text. For writing code examples inside
of rustdocs there's no way to avoid double-quotes though. Here's a
particularly heinous example from `core`:

    Splits a string into a vector of the substrings separated by a given string

    # Example

    ~~~
    assert [\"\", \"XXX\", \"YYY\", \"\"] == split_str(\".XXX.YYY.\", \".\")
    ~~~

The `rustdoc` tool understands what to do with `doc` attributes on crates,
items, enum variants, iface methods and impl methods. `doc` attributes found
in other places are silently ignored.

In future versions there will likely be other places to put rustdocs, like
inside comments or [here docs][1].

[1]: https://en.wikipedia.org/wiki/Here_document

### The rules of rustdoc

Though rustdoc imposes very few rules on what goes into a rustdoc, it does
have two rules for processing documentation that are important to know.

The first is _the summary rule_, under which rustdoc will attempt to use the
first sentence of the rustdoc as a "summary" of the documentation. This
summary is then used in the documentation index. Rustdocs are often formulated
like

    Get the next victim

    Get the next victim from the holding chambers. If there are no futher
    victims in the holding chambers then return `none`.

The documentation states in a few words what a function does, then reiterates
and expands on that. This is because the first line is "the summary". Another
common variant is to just make the first sentence short and simple:

    Get the next victim. If there are no further victims in the holding
    chambers then return `none`.

In this case rustdoc will use the text up to the first period as the summary.
There is an arbitrary limit on the length of a summary so make them brief. If
the first sentence is too long then rustdoc will not create a summary.

The second rule is _the header rule_.

Headers in rustdoc should be indicated with a single "#". Though markdown
supports multiple ways to write headers and multiple header levels, rustdoc
uses headers to interpret the structure of docs and currently the only
header markup that rustdoc understands is "#". This should be eased in
the future.

    # This is a valid rustdoc header

    #### Please don't do this

    Probably should avoid this too
    ==============================

### Recommended conventions for rustdocs

Even though rustdocs are just markdown, we still need to agree on some
conventions. Here is the official style used by the core and standard libraries.

The first sentence should describe, succinctly, what the item does. When
that is not sufficient to describe the behavior of the item then prose should
be used to fill out the description. It should discuss both what and why
the item does what it does, relevant inputs and outputs, and should _always_
mention any conditions under which the code might fail.

At some level of documentation complexity, simply writing prose about all
the possible inputs, outputs, effects, etc. gets unwieldy, and the document
should be broken into headered sections. Standard headers are "Arguments",
"Return value", "Failure", "Example", "Safety notes", and "Performance notes".

The argument section should be written as a bulleted list with the following form

    # Arguments

    * `s` - The input string
    * `n` - The starting index

### Operating the rustdoc tool

The simplest way to use rustdoc is by just passing it the crate file
to document, `rustdoc core.rc`. This will output html documentation
in the current directory.

The generated HTML expects to find a stylesheet called `rust.css` but
does not provide it. The one used for all of rust's documentation,
including the tutorial and manual, can be found in the rust repo under
"src/doc".  Eventually rustdoc will provide the sheet.

Rustdoc has a few other options. `--output-format` allows you to output
plain markdown, and `--output-style` can be used to generate all the
documentation on a single page (by default each mod gets its own page).

    Usage: rustdoc [options] <cratefile>

    Options:

        --output-dir <val>     put documents here
        --output-format <val>  either 'markdown' or 'html'
        --output-style <val>   either 'doc-per-crate' or 'doc-per-mod'
        --pandoc-cmd <val>     the command for running pandoc
        -h                     print help

### Other tidbits and limitations

Sometimes you don't want to advertise your public API. In that case you can
use the `#[doc(hidden)]` attribute and rustdoc will pretend the thing
doesn't exist.

There are many flavors of markdown, and rustdoc uses [Pandoc][1]. As such rustdoc
is subject to any Pandoc quirks or extensions. In general I recommend sticking
to pure markdown in most cases, with the exception of code blocks. For code
blocks using tilde-fences is the most reliable way of getting the correct
output. Using indented blocks works but is prone to ambiguities. Backtick
fences, like on github, do not work.

    # This works

    ~~~
    code here
    ~~~

    # This usually works

        code here

    # This does not work

    ````
    code here
    ````

[1]:http://johnmacfarlane.net/pandoc/

