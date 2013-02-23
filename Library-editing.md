# Library-editor's guide

The modules in the Rust standard library are of widely varying quality, completeness and coherence. This page provides guidelines and a process to aspiring "library editors" who want to _substantially_ improve, complete and standardize library modules. That is, those who want to take some degree of ownership over planning, organizing, editing and reviewing part of the Rust standard libraries.

If you have a small edit to make (a minor bug-fix or API-addition) or do not wish to have an ongoing organizational role in terms of editing libraries, this page is probably not relevant to you. Open a pull request and a library editor or project reviewer will take a look.

If this page _is_ relevant, we ask that you **please follow the guidelines in this page** when structuring your work. Library work has tendencies towards accidental duplication of effort, repetition of effort, and multiple conflicting implementations or attempts. Our goals with these guidelines are:
  - to minimize the unfortunate tendencies of library work stated above
  - to expedite approval of library work by setting expectations and authorizing new "library editors" to review changes for automated merging
  - to give library editors a sense of structure to work within and criteria to evaluate one another's work against

There are two kinds of library-editing work to consider:

  * "General" work of improving code quality across all (or many) modules, regardless of topic.
  * "Topical" work related to a specific module (or group of modules), informed by its area of focus.

For each type of work, we provide a sequence of steps which you should take if you want to work on a library. Following these steps will improve the odds of others in the community noticing your work, providing timely and relevant input, cooperating rather than working across-purposes, and producing results that get merged into our repository easily.

## General cross-module cleanup

There are many quality issues (interface consistency, obsolete idioms, non-conformance to the module edit criteria) that require many small patches applied over many different modules. These should  be addressed by the following steps:

  * Announce your intention to produce guidelines on the issue.
  * Discuss for 1 week on mailing list to solicit input from others on the issue.
  * Make a wiki page describing instances of the issue and how to solve them, collecting together point-form comments from the mailing list discussion.
  * Request the core project members approve of this cleanup task. It will get discussed and either approved or rejected at the next weekly meeting.

Once approved-of, a category of library cleanup can be used to expedite pull requests. In particular, put "cleanup:" in the title of the pull request and address _only_ a project-approved cleanup task in your pull request, and library editors can approve such changes for automated merging.

## Topical module-improvements

When you want to work on one or more _particular_ modules, due to wanting better library code for the topic the module is concerned with, we ask that you follow a relatively structured process involving research, consultation and (relatively) objective criteria. There are 5 main steps to follow:

  * Announce your intention on the mailing list, make a page on the wiki. Discuss on the mailing list for 1 week.
  * Research relevant standards (see below).
  * Research relevant libraries in other languages (see below).
  * Write or reorganize the modules in question based on the library criteria given in this page (see below).
  * Get another library-editor (or committer / reviewer) to review your module.

### 1. Announce your intention

  - Make a copy of the [[module edit plan template|Lib template]] and add it to the **Active Module Edit Plans** section of [[Libs]].
  - Send an email to the mailing list stating your intention to assume a library-editing role.
  - Put a link to the email in the template and note the date.
  - Discuss for _one week_ with the mailing list. Free-form discussion. Fish for ideas. See who else is interested.
  - **During this period, accept the possibility of veto**. If there is strong disagreement that the work you're proposing live in the standard library, you should probably keep it in an external package.
  - Assuming there is general consensus on the desirability of the module: make a short list of salient people and ideas that came up during that week in your module edit plan.

### 2. Survey existing standards and techniques

  - You can start this during the week of free-form discussion.
  - Go through any of the following standards organizations looking for documents pertaining to your module:
    - http://standards.iso.org/ittf/PubliclyAvailableStandards/index.html
    - http://standards.ieee.org/findstds/index.html
    - http://www.ecma-international.org/publications/standards/Standard.htm
    - http://www.apps.ietf.org/rfc/stdlist.html
    - http://www.unicode.org/reports/
    - http://www.w3.org/standards/
    - http://www.oasis-open.org/standards
    - http://en.wikipedia.org/wiki/Single_UNIX_Specification
    - http://en.wikipedia.org/wiki/Open_format
    - http://en.wikipedia.org/wiki/Free_file_format
    - http://en.wikipedia.org/wiki/Cryptography_standards
  - Go through any of the following lists looking for _techniques_ related to your module:
    - http//xlinux.nist.gov/dads/
    - http://en.wikipedia.org/wiki/List_of_algorithms
    - http://en.wikipedia.org/wiki/List_of_data_structures
    - http://en.wikipedia.org/wiki/List_of_file_formats
  - If you find such standards or techniques, _read them_.
  - Place links to such standards in your module edit plan.
  - Summarize in point-form the recommendations, leading techniques, industrial or academic standards you intend to follow or deviate from, and why. If no relevant standards exist, note this fact.

### 3. Survey existing languages

  - You can start this during the week of free-form discussion.
  - Look through at least 4, preferably 5 or 6 of the following other language ecosystems and standard libraries for libraries on the same topic:
    - C: 
      - http://www.cplusplus.com/reference/clibrary/
      - http://www.gnu.org/software/libc/manual/html_mono/libc.html
    - C++:
      - http://en.cppreference.com/w/cpp
      - http://www.boost.org/doc/libs/1_53_0/libs/libraries.htm
	- C#:
      - http://msdn.microsoft.com/en-us/library/aa388745.aspx
    - D:
      - http://dlang.org/phobos/index.html
    - Java:
      - http://docs.oracle.com/javase/7/docs/api/
    - Scala:
      - http://www.scala-lang.org/api/current/index.html
      - http://www.scala-lang.org/node/1209
    - Go:
      - http://golang.org/pkg/
      - https://code.google.com/p/go-wiki/wiki/Projects
    - Haskell:
      - http://www.haskell.org/ghc/docs/latest/html/libraries/index.html
      - http://hackage.haskell.org/packages/archive/pkg-list.html
    - OCaml:
      - http://caml.inria.fr/pub/docs/manual-ocaml-4.00/libref/Pervasives.html
      - http://caml.inria.fr/pub/docs/manual-ocaml-4.00/manual035.html
    - Ada:
      - http://www.ada-auth.org/standards/12rm/html/RM-A.html
    - Scheme:
      - http://srfi.schemers.org/
      - http://planet.racket-lang.org/
      - http://wiki.call-cc.org/chicken-projects/egg-index-4.html
    - Common Lisp:
      - http://www.lispworks.com/documentation/HyperSpec/Front/Contents.htm
      - http://clocc.sourceforge.net/
    - Python:
      - http://docs.python.org/3.2/py-modindex.html
      - http://pypi.python.org/pypi
    - Ruby:
      - http://www.ruby-doc.org/stdlib-1.9.3/
      - https://rubygems.org/gems
  - Place links to relevant source code, library or class names.
  - Summarize in point-form the following:
    - Structures and functions commonly appearing
    - Variations on implementation seen
    - Pitfalls and hazards associated with each variant
    - Relationship to other libraries and/or abstract interfaces

### 4. Write or edit the module

Modules should generally aim to satisfy these criteria:
  - Adequate surveying of standards has been done and documented
    - Put a doc-comment discussing such standards in the module.
    - Conform to any surveyed standards / leading techniques when possible.
    - **Avoid any patented techniques, and do not copy nonfree code**
  - Intersect, don't union, APIs found in other languages. When in doubt, do less at first, grow later.
  - Separate "one best variant" of an API from "several optional variants"
    - Give the most discoverable / obvious name to the "one best variant", give others less obvious names or place in submodules.
  - Priorities: docs > correctness > simplicity > speed.
  - Include `#[test]` based unit tests:
    - Those from from spec documents or behavior in other languages.
    - Randomly generated tests.
    - Hand-written tests.
  - Include `#[bench]` based benchmarks if possible
  - Minimize mutable state.
  - Minimize unsafe code.
  - Minimize heap allocation.
  - Minimize size and complexity of traits.
  - Minimize non-portable or platform-specific code.
  - Separately implement Owned variant and Managed variant when reasonable forms of both exist.
  - Similarly (possibly same as above), implement mutable/freezable variant separately from persistent/functional
  - Reuse other parts of standard libraries when possible.
  - Factor out common traits when >2 implementations exist.
  - Follow [[naming conventions|Lib naming conventions]]

### 5. File a pull request and ask another library editor or core developer for review

The reviewer should be reminded to evaluate your module on the criteria above. If a module is satisfactory in terms of these criteria, it should be merged.
