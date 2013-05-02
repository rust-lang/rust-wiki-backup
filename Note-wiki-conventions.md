The github wiki system is generally good but suffers from a couple weak points.

* No built-in category, tagging or namespace system
* No search

This makes it tricky to navigate. On the upside, it's written in simple markdown and can be accessed [through git](_access). This makes it very easy to keep manually organized, in the local filesystem.

A couple important points to keep in mind when editing and, particularly, naming pages:

* Please use markdown, not one of the many other markup languages GitHub provides.
* Let the filename be the title. Do not put any initial headers in pages, as github will (in some places) use that as the title rather than the filename. The filename is what you make internal wiki-links to (with dashes turned into spaces). Just use that name.
* We _manually maintain_ a category list in the file `_Sidebar.md`, which turns into the sidebar on all pages.
* Prefix every non-navigation page name with a category name. Crude, but it works. So when you make a developer note about foo, call it `Note-foo.md`. If it makes you uncomfortable, squint a bit and imagine the category name as a directory name.
