# Existing implementations

- [.NET](http://msdn.microsoft.com/en-us/library/system.io%28v=vs.71%29.aspx)
- [Go](http://golang.org/pkg/io/) - many fine-grained interfaces
- [Plan 9 BIO](http://swtch.com/usr/local/plan9/include/bio.h) - tidied up / simplified stdio
- [Python 3 IO](http://docs.python.org/3.2/library/io.html) - includes text/binary division
- [Common Lisp streams](http://www.lispworks.com/documentation/HyperSpec/Body/c_stream.htm) - very composable, possibly over-engineered
- [D stdio](http://dlang.org/phobos/std_stdio.html) - stdio-like
- [C++ iostreams](http://www.cplusplus.com/reference/iostream/) - Probably don't want to take a lot of influence from C++
- [Haskell System.IO](http://www.haskell.org/ghc/docs/latest/html/libraries/base-4.6.0.1/System-IO.html)

# Types of I/O streams

- File
- TCP
- UDP
- Memory buffer
- Strings
- Compressors/decompressors
- Encryption

