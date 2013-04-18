Module editing plan template

## 1. Announcement to mailing list

  - Proposed editor: _your name_
  - Date proposed: _date of proposal_
  - Link: _link to email_

###  Notes from discussion on mailing list

- [API sketch](https://gist.github.com/brson/5399629)

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
- [C (Plan 9) bio](http://swtch.com/usr/local/plan9/include/bio.h) [man](http://man.cat-v.org/plan_9/2/bio) - tidied up / simplified stdio
- [D stdio](http://dlang.org/phobos/std_stdio.html) - stdio-like
- [Go](http://golang.org/pkg/io/) - many fine-grained interfaces
- [Python 3 io](http://docs.python.org/3.2/library/io.html) - includes text/binary division
- [Common Lisp streams](http://www.lispworks.com/documentation/HyperSpec/Body/c_stream.htm) - very composable, possibly over-engineered
- [Haskell System.IO](http://www.haskell.org/ghc/docs/latest/html/libraries/base-4.6.0.1/System-IO.html)
- Ada [streams](http://www.ada-auth.org/standards/12rm/html/RM-13-13-1.html) [streamIO](http://www.ada-auth.org/standards/12rm/html/RM-A-12-1.html) [sequentialIO](http://www.ada-auth.org/standards/12rm/html/RM-A-8-1.html)
- Lua [LTN12: filters sources and sinks](http://lua-users.org/wiki/FiltersSourcesAndSinks) as used in [LuaSocket](http://w3.impa.br/~diego/software/luasocket/ltn12.html)

### Code examples

* [http://pleac.sourceforge.net/]
* [Ruby tutorial](http://ruby.bastardsbook.com/chapters/io/)

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

##### Error handling

* Return values - requires explicit checks, lots of noise
* Exceptions - results in nicer looking code, error handling at arbitrary granularity

#### Pitfalls and hazards associated with each variant
#### Relationship to other libraries and/or abstract interfaces

## 4. Module writing

  - Pull request: _link to bug_

### Additional implementation notes

#### Error handling

I/O is an area where nearly every operation can result in unexpected errors. It needs to be convenient to use I/O on the non-error path while also possible to handle errors efficiently.

Some use cases:
  * I'm hacking together a quick prototype or script that does a bunch of file manipulation. Thinking about all the possible ways I/O could fail is not the point of this work and it's ok for the entire process to fail if something goes wrong. I don't want to be inconvenienced with handling errors.
  * I'm doing some performance critical I/O on a server and want to recover from an I/O failure in the most efficient way possible. I'm ok with writing extra error handling code.

In Rust our error handling options are 'conditions' and the `Result` type.

Conditions are events that can be raised, usually upon error, and optionally handled by dynamically-scoped 'condition handlers'. An unhandled condition results in task failure. Condition handlers are not like exceptions - they are handled at the site of the error, not after unwinding the stack. There is no way in Rust to 'catch' an exception. Conditions are not used widely and it's unknown how well they work in practice.

The `Result` type is a monad-like type that can either be `Ok(A)` or `Err(B)`. Calculations on `Result` can be chained together in various ways based on previous results. They are generally considered unwieldy.

##### A proposed error handling strategy

The I/O library will use a combination of conditions and fallable (Result, Option) or otherwise 'nullable' values (explained in detail later).

All I/O errors will raise a condition and so will, by default, trigger task failure. This removes most of the obligation for the programmer to think about error handling on the successful path.

When a condition is handled I/O libraries will essentially be expected to turn the current operation into a no-op, possibly returning a sensible null or error value, as appropriate each specific function. This behavior of always being able to return *some* value on error will allow errors to be recovered without creating new tasks, which is seen as expensive, and also problematic when managed data is involved.

TODO

##### Reader/Writer composition

TODO

##### Questions

* Is ReaderUtil/WriterUtil the proper abstraction?
* Do we need to consider string/character readers more carefully than mixins on Readers and Writers? For example, there are multiple options for handling newlines, and the ReaderUtil/WriterUtil trait doesn't have the state to store a flag.
* Need to think about iteration, `each_byte` etc. and how it relates to iteration elsewhere.
* How to avoid allocating strings/vectors unnecessarily? e.g. string/character readers should slice into the string/vector instead of constructing an entirely new string to read a line, but a file reader will need to allocate a vector/string to read a line. If the API is going to be same between such readers, then the functions return (or, call callbacks with) either `&str` or `~str`. For `&str`, a function that wants a `~str` will have to allocate (a second time, for a file reader); for `~str`, the string reader will always have to allocate even though many callbacks only need a `&str`. (Possible solution: use a trait that allows strings to be converted to the other types of strings which only allocates when necessary, i.e. just returns the original string for `~str` -> `~str`.)