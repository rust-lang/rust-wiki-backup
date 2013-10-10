These are the criteria used for prioritizing issues. Issues meeting these criteria should be nominated for triage and are generally tagged with `P` tags on the issue tracker and eventually scheduled for major feature-based releases. These were formerly the criteria for 'maturity milestones'.

## Well-defined

This milestone says nothing about the stability of the language; reaching it means that we have the _means to measure_ the stability of the language. Until we have passed this milestone, we cannot meaningfully describe the stability of the language.

### Language level

for each production in the grammar, we have written down (not necessarily committed-to forever, but written down, in a clear way):

  - unambiguous syntax
  - expansion
  - attribute processing
  - resolution rules
  - borrow checking and type checking rules
  - static evaluation
    - what static items are created on success
    - what static combinations represent errors
  - dynamic semantics
    - what values are constructed at runtime and when
    - which actions represent failure or other types of error
  - code generation and link-time constraints and definitions
  - miscellaneous implementation requirements
  - miscellaneous implementation permissions / room for variability
  - miscellaneous implementation advice or notes
  - an example of the production

In addition, we have (at least for the grammar and the executable example, if not the type rules) the ability to extract and check the current compiler against the spec. This means parsing using the grammar and compiling-and-running the example(s).

### Tool level

For each tool in the language tool-suite, there exists:
  - a manual page
  - a mechanism for running an integration test on the tool

### Library level

For each library under consideration (core, std, etc.), there exists:
  - A list of symbols (including types) that exist, or a way to get it
  - A mechanism for generating a per-symbol smoketest that confirms the existence of the symbol, type of the symbol, and (if static) the value of the symbol. This mechanism will eventually form the basis of (simple) backward-compatibility testing.

### Platform target level

For each supported target, there exists:
  - Enough machinery to execute the testsuite and measure its success

## Backwards compatible

This is a subjective milestone that says that, given the ability-to-measure achieved in the "well defined" milestone, we are comfortable making a long-term support commitment to downstream users (servo in particular) to keep the set of symbols, definitions and passing-tests from contracting. We may add-to, clarify or enhance them, and we may make a new branch that deletes them, but at least on _some_ long-term-supported branch, we will not delete them or cause the set to contract or change existing parts.

This still leaves some leeway for "fixing bugs" in existing definitions and judgment calls made on whether a change represents a "bug fix" or a "breaking change".

## Feature complete



The purpose of this milestone is to measure the sense of "nothing missing" in the standard libraries, tool suite, target coverage and language semantics. That is, for anything reasonable that someone would want to do with a first-major-revision language, there's some reasonable way to do it with this language.

This milestone is reached when we would be comfortable potentially deferring all subsequent additions to APIs, tool options and behavior and language features on the stable branch, and only doing fixes and performance work.

("Potentially" used here because it's reasonable to backport or land any backward compatible features into the stable branch; this milestone is merely a marker of not needing any of those features on a stable branch for it to be useful in production)

This milestone also does not say anything about performance or fault levels.

## Well-covered

This milestone denotes a stronger set of measurement mechanisms than the "well defined" milestone, though it's similar in spirit: it means we have sufficient mechanisms to measure the _detailed_ correctness and performance of the language and libraries, not just the rough-cut versions.

it does not, as usual, define the actual correctness or performance targets themselves. Merely the ability-to-measure.

### Language level

For each grammar production, and for each specification-paragraph in the grammar production, there is a test in the testsuite that checks conformance, or a documented decision that testing that paragraph is not possible or desirable.

Since these tests are more extensive (and less-illustrative) than "example" code in the documentation, they should be out-of-line from the spec.

### Library level

For each listed public API symbol, there exists:

  - documentation of the symbol
  - an execution unit test that meaningfully tests the symbol's documentation, or a decision that testing the documentation is not possible or desirable
  - possibly: measurement of line-coverage or path-coverage to a chosen level
  - possibly: a unit benchmark or a documented decision that the symbol is not possible or desirable to benchmark

### Tool level

For each tool, there is an integration test for each tool/option combination in the manual page that confirms the end-to-end operation of the tool as documented, or a documented decision that testing is either not possible or undesirable.

## Production ready



The purpose of this milestone is to represent a high degree of confidence the language's suitability for industrial use due to a high level of measured correctness and performance in the tests.

In other words, for each of the aspects described in the "well covered" milestone, and for each supported target platform, test success and benchmark performance has reached a level deemed suitable for production use in environments with substantial business consequences for major defects or low performance. Where possible, performance benchmarks are linked to equivalent benchmarks in competitive languages, and rust is sufficiently close to equivalent in performance.
