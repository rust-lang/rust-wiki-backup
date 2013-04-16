Module editing plan template

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

- There are some

## 2. Research of standards and techniques

  1. Standard: _standard_
    - _link to docs_
    - ...

### Summary of research on standards and leading techniques
#### Relevant standards and techniques exist?
#### Those intended to follow (and why)
#### Those intended to ignore (and why)

## 3. Research of libraries from other languages

- [.NET](http://msdn.microsoft.com/en-us/library/system.io%28v=vs.71%29.aspx)
- Java [io](http://docs.oracle.com/javase/6/docs/api/java/io/package-summary.html) [nio](http://docs.oracle.com/javase/6/docs/api/java/nio/package-summary.html)
- [C / C++ stdio](http://www.cplusplus.com/reference/cstdio/)
- [C++ iostreams](http://www.cplusplus.com/reference/iostream/) - Probably don't want to take a lot of influence from C++
- [C++ boost iostreams](http://www.boost.org/doc/libs/1_53_0/libs/iostreams/doc/index.html)
- [C (Plan 9) bio](http://swtch.com/usr/local/plan9/include/bio.h) - tidied up / simplified stdio
- [D stdio](http://dlang.org/phobos/std_stdio.html) - stdio-like
- [Go](http://golang.org/pkg/io/) - many fine-grained interfaces
- [Python 3 io](http://docs.python.org/3.2/library/io.html) - includes text/binary division
- [Common Lisp streams](http://www.lispworks.com/documentation/HyperSpec/Body/c_stream.htm) - very composable, possibly over-engineered
- [Haskell System.IO](http://www.haskell.org/ghc/docs/latest/html/libraries/base-4.6.0.1/System-IO.html)
- Ada [streams](http://www.ada-auth.org/standards/12rm/html/RM-13-13-1.html) [streamIO](http://www.ada-auth.org/standards/12rm/html/RM-A-12-1.html) [sequentialIO](http://www.ada-auth.org/standards/12rm/html/RM-A-8-1.html)

### Summary of research from other languages:
#### Structures and functions commonly appearing

##### Types of I/O sources and sinks

- File
- TCP socket
- Unix domain socket
- Pipe
- UDP
- Memory buffer
- String
- Formatting + formatted values
- Serialization

##### IO adaptors / filters / converters

- Pattern matching and replacement
- Character encodings
- Compression
- Encryption

#### Variations on implementation seen
#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Additional implementation notes

  - _note_
  - _note_
  - _note_