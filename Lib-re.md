## 1. Announcement to mailing list

  - Proposed editor: Devin Jeanpierre
  - Date proposed: May 04, 2013
  - Link: https://mail.mozilla.org/pipermail/rust-dev/2013-May/004017.html

###  Notes from discussion on mailing list

Note that the majority of the discussion was had over IRC, and not the mailing list. Logs exist, but anything relevant is summarized in this document.

  - Consider previous attempt description: https://github.com/mcpherrinm/rerust/blob/master/Design.txt

## 2. Research of standards and techniques

  1. Standard: Unicode Technical Standard #18: UNICODE REGULAR EXPRESSIONS
    - http://www.unicode.org/reports/tr18/
  2. Standard: Open Group Base Specification (Regular Expressions)
    - http://pubs.opengroup.org/onlinepubs/009695399/basedefs/xbd_chap09.html
    - There are other specifications for UNIX/POSIX, not sure which are preferred (and free)
  3. Standard: ECMA-262 (RegExp Objects)
    - http://www.ecma-international.org/ecma-262/5.1/#sec-15.10
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
 ROBDD TNFA Simulation               | Backreferences, assertions?              | ???      | ???
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

* **TODO:** describe DFA compilation, Thompson NFA search, RE2 "no-backtrack" backtracking, preprocessing costs, and best-case time complexity. Describe OBDD-TNFA performance. Determine if OBDD-TNFA can do assertions using the memoization approach, or using a new approach utilizing features of OBDD-TNFA. Determine if the algorithm for recovering matches with OBDD-TNFA can be used to recover assertion information, solving the O(m) assertion problem.

#### Relevant standards and techniques exist?

Yes.

#### Those intended to follow (and why)

The ECMA 262 standard should be followed. See discussion on implementations for details as to why.

As far as implementation techniques go, there's two fundamental groups of people: those who are willing to hand-tune the regexp and don't want extra overhead, and those who won't or can't and want acceptable worst-case performance. It is reasonable to support both use cases by having a first-class implementation of both backtracking search (tunable, low overhead in best case) and Pike VM style implementation (excellent worst case time).

In order to allow for the "excellent worst-case time" implementation to be as efficient as possible, RE2 implements multiple algorithms handling special cases of regular expressions. This can be considered a "stretch goal", in that the people that really care might want to use backtracking search, but it is valuable to make the algorithmically good solution also as close as possible to the solution that is best everywhere.

**FIXME:** Discuss assertions.

#### Those intended to ignore (and why)

The POSIX regexp standard is not compatible with ECMA 262, and will be ignored.

The Unicode regexp standard is intended for those designing new PCRE variants, and will be ignored in favor of doing whatever ECMA 262 does. However, any implemented extensions to ECMA 262 should draw on the Unicode standard rather than making things up.

For the time being, ROBDD NFA simulation is too new and untested to rely on for good performance.

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
  8. Language: ECMAScript/JS
    - http://www.ecma-international.org/ecma-262/5.1/#sec-15.10
    - https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Regular_Expressions

### Summary of research from other languages:
#### Structures and functions commonly appearing

FIXME: note difference between "streaming" style APIs and the more python-ish APIs

- Separate type for compiled regexp, separate type for result of a search operation
  * result.start, result.end, result.groups (or equivalent)
- function for compiling a regexp to a compiled regexp, with options:
  * ignore case
  * multiline (^/$ match start/end of line)
  * bytes vs unicode
  * verbose/freeform mode (where whitespace is ignored unless escaped)
- Function for searching; sometimes searching with anchors (e.g. python's re.match vs re.search; RE2's FullMatch vs PartialMatch)
  * Support for controlling area of string to search (but we can use slices)
- function for search/replace
- function for splitting text by a regexp

#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

- May suffice to implement string tokenization library from https://github.com/mozilla/rust/wiki/Note-wanted-libraries
- May be useful for implementing glob library from that same list of wanted libraries.
- Depends on core::unicode, want core::unicode to be improved for extra unicode features.

### Compatibility

It is too easy to write your own PCRE variant from scratch. Most likely, what we really want is to take some PCRE variant, and say "we're compatible with that" (maybe with a few clearly defined extensions and removals). If we do this, then we get the following benefits:

- Lower learning curve: if we're identical to a widely used PCRE variant, then people used to that variant need learn nothing new.
- Less documentation burden: if we bind ourselves to be compatible with some syntax/semantics, we can say "if you're unsure about something, check their docs". Especially if the other thing is a documented standard, as opposed to some module that might change its semantics backwards-incompatibly from underneath us.
- Less bikeshedding: it becomes harder to arbitrarily redefine things for silly and subjective reasons. An existing standard ties our hands, in a good way.

Therefore I think it is quite possible that we should be approximately 100% compatible with some implementation as far as the syntax and semantics of regular expressions are concerned. And then with a little thought about the various possibilities, ECMAScript/JavaScript stands out as something we should _definitely_ copy:

- It is a standard, with a stable and clearly documented syntax/semantics as a result. (Not to mention the billions of people that run code that relies on its semantics.)
- It has the most implementations of any PCRE variant. In fact, very few other PCRE variants have multiple implementations. This also assists stability, and means that the semantics are clearly enough explored for us to copy without too much trouble.
- It is perhaps the most widely used PCRE variant there is, but if not, it's certainly up there. As a result, lots of programmers are familiar with it.
- It is relatively simple, without too many crazy extensions. This makes the implementation easier (phew!)
- Considering its worldwide use across the web, we can safely assume its Unicode support is at least sufficient.
- There is a simply massive library of tests and benchmarks. Not only is this the regexp dialect with the most implementations, but it also has the fastest and most complex implementations (meaning excellent test banks), and the most intense performance competition between implementations (meaning excellent benchmarks to draw on).

A possible objection is that if we tie ourselves to ECMAScript, it may change backwards incompatibly and leave us in the dust, forcing us to either remain out of date, or adapt backwards incompatibly. However, as noted above, ECMAScript is _very_ widely used. Any backwards incompatible changes would be introduced in such a way as to not break existing ECMAScript code without a lot of time to prepare -- for example, in a new version of JavaScript, which has to be opted into by code. In that case, we can support it in a similar fashion, with a new version of regexps, which have to be opted into via a flag or option, with a similarly long period to allow code to adapt. We can probably trust the JS developers to be far more worried about backwards compatibility than Rust is, for the foreseeable future. If and when that changes, we can always elect to desynchronize from ECMAScript regexps.

Does this seem reasonable? :)

## 4. Module writing

  - Pull request: _NOT YET MADE_

### API
  
####re!() syntax extension

Pro:

- Compile-time syntax and type checking (can fail these: `re!("[")`, `let x, y = re!("(1)(2)(3)").fullmatch("123").groups()`)
- Compile to rust/LLVM bitcode at build time, don't even need to JIT at runtime.

Cons:

- Syntax extensions must be defined in the parser currently.

Issue: should it always work at compile time, or optionally fall back to runtime?

- Pro always:
  * Simplified API. We know that re!() will never fail, don't need to be concerned with that.
  * Predictability. We know that the work is done at compile time, so that at runtime there is no compilation overhead.

- Pro sometimes:
  * Simplified API. There is no separate API for runtime compilation, and making a change from compile-time to run-time regexps doesn't involve changing functions.

After some discussion on #rust, the consensus appears to be that re!() should only be compile time, and there should be a re() only for run-time.

IMO a parse!() would be useful for the cases where compilation cannot occur statically, but parsing can. It can at least verify the regexp syntax statically.

#### Separation of parsing from compiling

Suppose we separate parsing from compiling. Then there are four situations:

- A builtin parser and compiler is used. Compile-time syntax and type checking is possible, as is compile-time regexp compilation.
- A builtin parser and custom compiler is used. Compile-time syntax (and type checking, if the compiler is "reasonable" and preserves groups and such) is possible, but regexp compilation is not. (Compile-time parsing is also done, but if this is slow enough to care about, we've done something wrong.)
- A custom parser and builtin compiler is used. No compile-time work can be done.
- A custom parser and compiler is used. No compile-time work can be done.

If parsing and compiling are unified, then the second case is lost. This is in addition to the usability losses of not having separated interfaces for parsing and compiling. Food for thought!

#### Notion of a regexp "engine"

Suppose that we have the following implementation approaches:

* Backtracking search
* Memoized backtracking search
* TNFA search
* NFA search
* DFA search
* Backtracking-free "backtracking" search

And suppose that the following options may apply:

* Find PCRE-style match
* Find shortest match.
* Find longest match.
* Only discover if a match exists
* Ignore case (and other normal regexp flags)
* Forbid assertions
* Run assertions by running the same search type as a subroutine
* Run assertions by running the same search type as a subroutine, but with results memoized

It's probably best to have engines (as in the first list) with options (from the second list), considering that most of the time every option applies to every of the search types.

It's also possibly best to have some means of automatically selecting an engine based on the regex or such, so a special "Auto" engine which selects the most efficient (in algorithmic terms) engine that can run the regex.

#### WIP notes

- All-in-one `re!()`: https://gist.github.com/Kimundi/5543809
- Compiletime only `re!()`: https://gist.github.com/Kimundi/5553204

### User stories

* I am the author of an online regexp sandbox. I do not want my server to be DDOSed.
  * Concession: Tagged NFA (Pike VM, etc.) simulation (non-exponential worst-case time)
  * Concession: Measurability of number of states of NFA (to estimate cost of search)
  * Concession: Predictability of size of NFA based on regexp? (only necessary if NFA space usage can grow exponentially in the regex length; instructions like a Repeat instruction which has exponential _state_ but not exponential _space_ may make this unnecessary).
  * Concession: Way to bound time used by regexp (e.g. running in a subtask)? Needs investigation.
* I am a regexp power user. I want to be able to fine-tune a regexp to perform well, because I know what I am doing.
  * Concession: Backtracking search (which has low overhead and quick times in best case)
  * Concession: Various regexp operators that tailor backtracking search process (e.g. atomic grouping)
* I'm just a regular guy hacking up a file in a weird format, and I don't care about any of that stuff. Just don't take too long.
  * Would defaulting to Tagged NFA simulation make this person happier than defaulting to backtracking search?
* I an writing a service, and would like to validate user input using a regex.
  * Different from sandbox case; regex is trusted, input is not. Probably the same advice still applies.

#### Notes

While usually the power user / safety user would use different libraries, there's no particular reason both their needs can't be satisfied by one library. Plus, sometimes even with the overhead, Tagged NFA based search may be faster than backtracking search for some problems, no matter how much the power user tries to optimize the regex for backtracking search.

### Additional implementation notes

  - _note_