## 1. Announcement to mailing list

  - Proposed editor: Devin Jeanpierre
  - Date proposed: May 04, 2013
  - Link: https://mail.mozilla.org/pipermail/rust-dev/2013-May/004017.html

###  Notes from discussion on mailing list

  - _NOT YET POSTED_

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

### Summary of research on standards and leading techniques

#### Classification of techniques.

Pike VM is an extension of Thompson NFA to handle submatch extraction. We care about that, so let's ignore Thompson NFA as a separate topic.

Pike VM and Laurikari TNFA are apparently similar techniques. I am not very familiar with TNFA currently, so for now I will lump them together.

#### Summary of techniques

(Performance details are given where m is the size of the regex, and n is the size of the string being matched.)

Backtracking search is the simplest to implement and has the least overhead when nothing goes wrong. However, in the presence of a few regex operators and bad input, it also has the worst performance, with worst case O(2^n) time and O(n) space.

Pike VM / TNFA have more overhead, but worst-case O(nm) time and O(m) space complexity. They can't implement backreferences, and may not be able to implement zero width assertions.

Memoized backtracking search is O(nm) time and O(nm) space. It can't implement backreferences, but can implement zero width assertions. It is not difficult to imagine combining this approach with Pike/TNFA so that the memo cache is only used for keeping track of zero width assertions, so as to get the benefits of both approaches.

It is also not difficult to imagine falling back to a backtracking implementation in the presence of backreferences, if those are to be supported.

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

### Additional implementation notes

  - _note_