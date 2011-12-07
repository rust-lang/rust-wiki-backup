Metadata is specified in the `.note.rustc` section of a crate. The metadata is in EBML format.

# Schema

*TODO: This is incomplete.*

The schema is given in pseudo-BNF. Some nonterminals are delimited by EBML tags; for these, the tag that surrounds the data is given in parentheses, for brevity. (For example, `foo (TAG_FOO) ::== bar` is really `foo ::== TAG_FOO bar END_TAG_FOO`.)

crate ::== attributes crate-deps tag-paths tag-items

attributes (TAG_ATTRIBUTE) ::== attribute*

crate-deps (TAG_CRATE_DEPS) ::== crate-dep*

paths (TAG_PATHS) ::== paths-paths paths-index

paths-paths (TAG_PATHS) ::== paths-data*

items (TAG_ITEMS) ::== items-data* items-index