## 1. Announcement to mailing list

  - Proposed editor: Devin Jeanpierre
  - Date proposed: May 04, 2013
  - Link: https://mail.mozilla.org/pipermail/rust-dev/2013-May/004017.html

###  Notes from discussion on mailing list

  - Consider previous attempt description: https://github.com/mcpherrinm/rerust/blob/master/Design.txt

## 2. Research of standards and techniques

  1. Standard: Unicode Technical Standard #18: UNICODE REGULAR EXPRESSIONS
    - http://www.unicode.org/reports/tr18/
  2. Standard: Open Group Base Specification (Regular Expressions)
    - http://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap09.html
    - There are other specifications for UNIX/POSIX, not sure which are preferred (and free)
  1. Technique: Thompson NFA
    - http://swtch.com/~rsc/regexp/regexp1.html
  2. Technique: Pike VM / Laurikari TNFA
    - http://swtch.com/~rsc/regexp/regexp2.html (VM, with submatch extraction)
    - http://swtch.com/~rsc/regexp/regexp3.html (tour of RE2, a production implementation of VM)
    - http://laurikari.net/ville/regex-submatch.pdf (TNFA, thesis)
    - http://laurikari.net/ville/spire2000-tnfa.ps (TNFA and compilation to TDFA)
  3. Technique: Backtracking Search
    - http://en.wikipedia.org/wiki/Backtracking (not a very hard algorithm, this is sufficient)
  4. Technique: Memoized Backtracking Search
    - Discussed in Russ Cox Regexp2 above ( http://swtch.com/~rsc/regexp/regexp2.html )
  5. Technique: OBDD NFA simulation:
    - http://link.springer.com/chapter/10.1007%2F978-3-642-15512-3_4 (without submatch extraction)
    - http://www.hpl.hp.com/techreports/2012/HPL-2012-215.pdf (with submatch extraction)
  6. Technique: Generalized Boyer-Moore (limited)
    - http://www.sciencedirect.com/science/article/pii/S0167642303000133

### Summary of research on standards and leading techniques

#### Classification of techniques.

Pike VM is an extension of Thompson NFA to handle submatch extraction, with identical performance characteristics for regexps that have no submatch extraction. Therefore, we can ignore Thompson NFA as a separate topic.

Pike VM and Laurikari TNFA simulation are apparently similar techniques. I am not very familiar with Laurikari's TFNA simulation algorithm currently, so for now I will lump them together.

#### Summary of techniques

 Technique                           | Features missing                         | time     | space
 ----------------------------------- | ---------------------------------------- | -------- | -----
 Backtracking Search                 | None                                     | O(2^n)   | O(n log n) ?
 Memoized Backtracking Search        | Backreferences                           | O(nm)    | O(n m log n) ?
 Pike VM / Laurikari TNFA simulation | Backreferences, assertions               | O(nm)    | O(m log n) ?
 OBDD TNFA Simulation                | Backreferences, assertions?              | ???      | ???
 Generalized Boyer-Moore             | Backreferences, assertions, groups (???) | O(nm)??  | ???

 **Note on features**: The list of missing features is only with the "standard" algorithm. There may be easy modifications to bring back some features (see the detailed summary for ideas).

 **Note on complexity**: `n` is the size of the input string, `m` is the size of the "pattern", specifically the regex in backtracking searches, and the automaton in automata search. This is somewhat misleading, because the worst-case automaton size is exponential in the size of the regex (consider `.{10}{10}{10}` which has 1000 states), but in practice this doesn't seem to be something people are concerned about.

#### Notes about techniques

* I didn't think too hard about space complexity. The `log n` component is for keeping track of match indices.

* Backreferences cannot be implemented efficiently unless P=NP.

* Whether or not assertions can be implemented in `O(m)` space with an automaton approach is AFAIK an open problem.

* It is possible to combine the memoization approach with automaton search to get assertions, but where a regexp with assertions uses something like `nm` space to store a record of what assertions match at what places (and with what result).

* It is also not difficult to imagine falling back to a backtracking implementation in the presence of backreferences, if those are to be supported.

* Generalized Boyer-Moore is interesting in that it is the only algorithm presented that has better best-case time complexity than backtracking search. In fact, Boyer Moore is frequently used instead of regexps because of its ability to skip large sections of text. However, I don't yet know exactly what features can be implemented under generalized Boyer-Moore.

* FIXME: describe DFA compilation, preprocessing costs, and best-case time complexity

#### Relevant standards and techniques exist?
#### Those intended to follow (and why)
#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: Perl
    - http://perldoc.perl.org/perlre.html
  2. Language: C (PCRE)
    - http://www.pcre.org/
  4. Language: C++ (RE2)
    - http://code.google.com/p/re2/
  5. Language: D
    - http://dlang.org/regular-expression.html
  6. Language: Python
    - http://docs.python.org/2/library/re.html
  7. Language: Common Lisp
    - http://weitz.de/cl-ppcre/

### Summary of research from other languages:
#### Structures and functions commonly appearing
#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

- May suffice to implement string tokenization library from https://github.com/mozilla/rust/wiki/Note-wanted-libraries
- May be useful for implementing glob library from that same list of wanted libraries.
- Depends on core::unicode, want core::unicode to be improved for extra unicode features.

## 4. Module writing

  - Pull request: _NOT YET MADE_

### API
  
####re!() syntax extension

Pro:

- Compile-time syntax and type checking (can fail these: `re!("[")`, `let x, y = re!("(1)(2)(3)").fullmatch("123").groups()`)
- Compile to rust/LLVM bitcode at build time, don't even need to JIT at runtime.

Cons:

- Syntax extensions must be defined in the parser currently.

### Additional implementation notes

  - _note_