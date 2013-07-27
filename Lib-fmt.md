Module editing plan template

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

  - ["RFC: User-implementable format specifiers w/ compile-time checks"](https://mail.mozilla.org/pipermail/rust-dev/2013-May/003999.html) 2013-05-03
  - ["Redesigning fmt!"](https://mail.mozilla.org/pipermail/rust-dev/2013-July/004961.html) 2013-07-26
  - _note_

## 2. Research of standards and techniques

  1. Standard: _standard_
    - _link to docs_
    - ...
  2. Standard: _standard_
    - _link to docs_
    - ...
  1. Technique: _technique_
    - _link to docs_
    - ...
  2. Technique: _technique_
    - _link to docs_
    - ...

### Summary of research on standards and leading techniques
#### Relevant standards and techniques exist?
#### Those intended to follow (and why)
#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

  1. Language: C++
    - [boost format](http://www.boost.org/doc/libs/1_53_0/libs/format/)
  2. Language: Java
    - [java.text.MessageFormat](http://docs.oracle.com/javase/1.4.2/docs/api/java/text/MessageFormat.html)
    - [java.util.Formatter](http://docs.oracle.com/javase/6/docs/api/java/util/Formatter.html)
  3. Language: Common Lisp
    - [Format](http://www.lispworks.com/documentation/HyperSpec/Body/22_c.htm)
  4. Language: Python 3
    - [Format strings](http://docs.python.org/3/library/string.html#formatstrings)
  5. Language: C/POSIX
    - [ICU MessageFormat](http://userguide.icu-project.org/formatparse/messages)
    - [Printf](http://pubs.opengroup.org/onlinepubs/9699919799/functions/printf.html)
  6. Language: C#/.NET
    - [Formatting types](http://msdn.microsoft.com/en-us/library/26etazsy.aspx)
  7. Language: Go
    - [go.fmt](http://golang.org/pkg/fmt/)
  8. Language: JS
    - [MessageFormat.js](https://github.com/SlexAxton/messageformat.js)


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