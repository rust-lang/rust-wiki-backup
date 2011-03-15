Request for comments—I've been trying to work on
[issue 244](https://github.com/graydon/rust/issues/labels/a.marijn#issue/244).
Since my understanding of the compiler internals isn't all that deep
yet, and everybody else on the team is asleep when I'm working, I
think it'd be useful to have some discussion in wiki format before I
start writing actual, possibly wrong-headed code. Please add any
comments or problems you can see in the text below. (Use a separate
paragraph, sign with your name at the end.)

***

(A crate will be a 'native' object file-ELF, Macho-O, COFF/PE—with one
or more sections inserted that describe the crate to Rustc.)

The crate meta information will be simple to store. Whether it'll be a
separate section or not hasn't been decided yet. I think it should
be—the code reading the meta information is not (yet) interested in
all the other stuff, so isolating that in a different section seems
sane.

***

The question is how to represent the other information, which will
help Rustc to

 * figure out which functions, constants, etc are exported from the
   crate.

 * type-check uses of these items.

 * (maybe) map these uses to linker symbols.

***

An initial idea (which, as far as I understand, is what rustboot does)
was to serialize the whole crate AST, leaving out the function bodies.
Then a 'use' causes this AST to be deserialized and run through most
of the compiler as usual, so that external definitons are treated just
like local ones.

The downsides to this approach seem to be:

* Inefficiency on big interfaces—a lot of information that was
  already known is reconstructed from a more verbose representation.

* It doesn't help with 'forwarding' exports—referring to X.a, where
  module X contains an import Y.a. See [issue 268](https://github.com/graydon/rust/issues/closed#issue/268). Rustboot
  currently doesn't do this at all. It could be kludged into this
  approach, of course.

***

Here is the alternative I came up with:

The crate descriptor section contains a table mapping paths (x.y.z) to
either a local def id, or a number that identifies another crate. Most
paths will refer to ids in the crate itself, but the crate will also
contain a table of crates it depends on, and a path may be 'forwarded'
to a name in one of these crates, which will cause the compiler to
repeat the lookup with the new name in the new crate. This table is
used by the resolver.

(Since the local def ids aren't stable across recompilation, we
unfortunately can't just refer to a (crate, def id) pair, but have to
include the name. We could use cryptographic hashes, but they'll
generally be bigger than the names, so that's probably pointless.)

The table will be structured so that one can quickly pick out just a
few paths, so that you only pay for the symbols you actually use
during compilation. Either a nested table (per path item) or a hash
table should do fine.

Next, there is a type table, a symbol table, and possibly more tables
(constants?) mapping the the crate's internal def ids to, in the first
case, serialized middle.ty.t values, and in the second case, linker
symbols. The typechecking pass will access the type table, the
translator will need both.

This would mean that linker symbols are determined by the compiler when it compiles the crate, and found in a table when compiling other crates that depend on it. This *seems* to map well to how middle/trans.rs looks up symbols by def_id—not much change would be needed there.

It a separate constant table is necessary, it should probably accommodate for both 'inlinable' constants (if we want to inline constants from other crates at all) and constants that live at a given symbol. Comments on what any tables that might be needed, and what they would contain, would be much appreciated.

***

It seems like it will be easy to have the typechecker and translator
return their internal tables along with their regular output, and use
these, after the regular compilation passes, to output this data.

On the input side, a crate loaded by the compiler in response to a
`use` clause will support the following operations:

 * Construct, given a crate id (to be used in ast.def_ids) and a
   filename. This will simply memory-map the file and read the crate
   table.
 
 * Lookup a path. Returns either ast.def_id or a forwarding (crate_file,
   original_name).

 * Lookup a type. Map from a the local id in a def_id to an ty.t.

 * Lookup a symbol. Map from local id to str.

All lookups can be cached, so that the other components can just
access it in any naive way they want without worry. This'll be
stateful, but conceptually referentially transparent, so we can just
auth it to be pure.
