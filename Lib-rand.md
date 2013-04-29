Generating random numbers, and sampling from random distributions.

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

  - [first discussion](https://mail.mozilla.org/pipermail/rust-dev/2013-April/003843.html)

## 2. Research of standards and techniques

  1. Standard: _standard_
    - _link to docs_
    - ...
  2. Standard: _standard_
    - _link to docs_
    - ...
  1. Technique: Generating random bits/numbers (i.e `u32` or `u64`)
    - http://en.wikipedia.org/wiki/List_of_random_number_generators
    - [ISAAC/ISAAC-64 RNG](http://en.wikipedia.org/wiki/ISAAC_%28cipher%29)
       - http://burtleburtle.net/bob/rand/isaacafa.html
       - ISAAC is the main RNG used in rust at the moment
  2. Technique: Testing quality of random numbers 
    - Very important! Extremely hard to tell if random numbers are "random enough" (a bug in an implementation, or a bad algorithm, can produce numbers that look random but aren't random enough for many purposes).
    - [Overview wikipedia article](http://en.wikipedia.org/wiki/Randomness_test)
    - [Diehard tests](http://en.wikipedia.org/wiki/Diehard_tests) (e.g. [dieharder](http://www.phy.duke.edu/~rgb/General/dieharder.php))
    - [TestU01](http://en.wikipedia.org/wiki/TestU01) including "Small crush", "Crush" and "Big crush"
    - [tests written by the creator of ISAAC](http://burtleburtle.net/bob/rand/testsfor.html)
    - Add a new make target "check-rngs" with a testsuite? 
  3. Technique: sampling from distributions
    - [Inverse transform sampling](http://en.wikipedia.org/wiki/Inverse_transform_sampling) (fully general)
    - [Ziggurat algorithm](http://en.wikipedia.org/wiki/Ziggurat_algorithm) (distributions with decreasing density functions)
    - [Box-Muller transform](https://en.wikipedia.org/wiki/Box%E2%80%93Muller_transform) and [Marsaglia polar method](https://en.wikipedia.org/wiki/Marsaglia_polar_method) (normal distribution, both are almost certainly inferior to the ziggurat algorithm)
    - ["A simple method for generating gamma variables" Marsaglia & Tsang 2000](http://dl.acm.org/citation.cfm?id=358414)

### Summary of research on standards and leading techniques
#### Relevant standards and techniques exist?
#### Those intended to follow (and why)
#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: C++
    - http://www.cplusplus.com/reference/random/
  2. Language: Python
    - http://docs.python.org/3.3/library/random.html
  3. Language: R (statistical language, so much broader random number support than necessary)
    - http://stat.ethz.ch/R-manual/R-patched/library/base/html/Random.html
    - http://stat.ethz.ch/R-manual/R-patched/library/stats/html/Distributions.html
  4. Language: Julia (also a statistical language)
    - https://github.com/JuliaStats/Distributions.jl
  4. Language: D
    - http://dlang.org/phobos/std_random.html (no support for sampling from distributions other than uniform, but many RNGs)
  5. Language: Go
    - http://golang.org/pkg/math/rand/
  6. Language: Erlang
    - https://mail.mozilla.org/pipermail/rust-dev/2013-April/003881.html
  4. Library: [GSL](https://www.gnu.org/software/gsl/manual/html_node/)
    - https://www.gnu.org/software/gsl/manual/html_node/Random-Number-Generation.html
    - https://www.gnu.org/software/gsl/manual/html_node/Random-Number-Distributions.html

### Summary of research from other languages:
#### Structures and functions commonly appearing
#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Additional implementation notes

  - _note_
  - _note_
  - _note_