[Travis CI Status](http://hiho.io/rust-ci/)

## Audio

See [[Computer Graphics and Game Development]].

## Collections

See [[Containers]].

## Compression

We want compression/decompression support for TAR and ZIP file formats.

* [erickt/rustzlib](https://github.com/erickt/rustzlib): libzlib bindings.
  * Status: last commit in December 2012
* [thestinger/rust-snappy](https://github.com/thestinger/rust-snappy): libsnappy bindings
  * Status: last commit in September 2013

## Computation

* OpenCL
  * [eholk/rust-opencl](https://github.com/eholk/rust-opencl): OpenCL bindings [<img src="https://travis-ci.org/eholk/rust-opencl.png?branch=master">](https://travis-ci.org/eholk/rust-opencl)
 
## Concurrency
 * task management, actor, OTP, [[Bikeshed mapreduce]], pools
 
## Crypto

Do not belong in std.

* [crypto](https://github.com/mozilla/rust/tree/master/src/libextra/crypto)
* [DaGenix/rust-crypto](https://github.com/DaGenix/rust-crypto): [<img src="https://travis-ci.org/DaGenix/rust-crypto.png?branch=master">](https://travis-ci.org/DaGenix/rust-crypto)
* [erickt/rustcrypto](https://github.com/erickt/rustcrypto): OpenSSL libcrypto bindings
  * Status: last commit in August 2013
  
## Database Access

Do not belong in std.

* NoSql
  * LevelDB
      * [lht/rust-leveldb](https://github.com/lht/rust-leveldb) (external)
  * MongoDB
      * [10gen-interns/mongo-rust-driver-prototype](https://github.com/10gen-interns/mongo-rust-driver-prototype) (external)
  * Redis
      * [mneumann/rust-redis](https://github.com/mneumann/rust-redis) (external)
* SQL
  * PostrgreSql
      * [sfackler/rust-postgres](https://github.com/sfackler/rust-postgres): [<img src="https://travis-ci.org/sfackler/rust-postgres.png?branch=master">](https://travis-ci.org/sfackler/rust-postgres)
  * Sqlite
      * [kud1ing/rustsqlite](https://github.com/kud1ing/rustsqlite): Sqlite3 bindings [<img src="https://travis-ci.org/kud1ing/rustsqlite.png?branch=master">](https://travis-ci.org/kud1ing/rustsqlite)
 
## Date and Time
  * [luisbg/rust-datetime](https://github.com/luisbg/rust-datetime)
  * [tedhorst/rust_datetime](https://github.com/tedhorst/rust_datetime)
  
## Encoding
* [Base64](https://github.com/mozilla/rust/blob/master/src/libextra/base64.rs)
* Cap'n Proto
    * [dwrensha/capnproto-rust](https://github.com/dwrensha/capnproto-rust):  [<img src="https://travis-ci.org/dwrensha/capnproto-rust.png?branch=master">](https://travis-ci.org/dwrensha/capnproto-rust)
* Character Encoding
    * [lifthrasiir/rust-encoding](https://github.com/lifthrasiir/rust-encoding)
* [grahame/rust-csv](https://github.com/grahame/rust-csv)
* HTML
    * [veddan/rust-htmlescape](https://github.com/veddan/rust-htmlescape):  [<img src="https://travis-ci.org/veddan/rust-htmlescape.png?branch=master">](https://travis-ci.org/veddan/rust-htmlescape)
* [JSON](https://github.com/mozilla/rust/blob/master/src/libextra/json.rs)
* [ProtocolBuffers](https://github.com/stepancheg/rust-protobuf) (external)
* Thrift
  * see https://github.com/mozilla/rust/issues/1677
* [tnetstring](https://github.com/erickt/rust-tnetstring) (external)
* XML
   * [bjz/sax-rs](https://github.com/bjz/sax-rs): bindings to libxml2's SAX parser [<img src="https://travis-ci.org/bjz/sax-rs.png?branch=master">](https://travis-ci.org/bjz/sax-rs)

## Graphics

See [[Computer Graphics and Game Development]].

## GUI
 * [rust-cocoa](https://github.com/mozilla-servo/rust-cocoa) (external)
 * [wxRust](https://github.com/kenz-gelsoft/wxRust) (external)
	  
## IO
 * AIO, SIO, stdio
 * [filesystem](https://github.com/mozilla/rust/blob/master/src/libstd/os.rs)
 * [path manipulation](https://github.com/mozilla/rust/blob/master/src/libstd/path.rs)
 * [<> or fileinput](https://github.com/mozilla/rust/blob/master/src/libextra/fileinput).
 * timers
 
## Localizability
 * one aspect of L10n is to map a key to a text, based on the current locale (eg Java's [ResourceBundle](http://docs.oracle.com/javase/7/docs/api/java/util/ResourceBundle.html) or [GNU gettext](http://www.gnu.org/software/gettext/))
 * another aspect is to format a string based on the current locale (eg Java's [MessageFormat](http://docs.oracle.com/javase/7/docs/api/java/text/MessageFormat.html))
 * See [issue #4630](https://github.com/mozilla/rust/issues/4630)
 
## Mathematics
  * [bjz/lmath-rs](https://github.com/bjz/lmath-rs) (external)

## Misc
* low-level OS services
* Simple search on a filesystem (eg Ruby's [glob](http://ruby-doc.org/core-2.0/Dir.html#method-c-glob))
* unit testing
* FFI, ctypes
* dlopen, os processes
* standard predicates
 * text, numeric, sorted
* error-trapping wrappers, in-place task?
 * Consistent error handling
* quotas, accounting
* reflection

## Networking
* HTTP
    * [rust-http](https://github.com/chris-morgan/rust-http):  [<img src="https://travis-ci.org/chris-morgan/rust-http.png?branch=master">](https://travis-ci.org/chris-morgan/rust-http)
* [URI/URL](https://github.com/mozilla/rust/blob/master/src/libextra/url.rs)
* [UUID](https://github.com/mozilla/rust/blob/master/src/libextra/uuid.rs)
* GUID
* ZeroMQ
  * [erickt/rust-zmq](https://github.com/erickt/rust-zmq):  [<img src="https://travis-ci.org/erickt/rust-zmq.png?branch=master">](https://travis-ci.org/erickt/rust-zmq)
    * How will libs like this be integrated with regular nonblocking io?
    * ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?
 
## Random Numbers
* [random](https://github.com/mozilla/rust/blob/master/src/libcstd/rand.rs)

## Regular Expressions
See https://github.com/mozilla/rust/issues/3591
* [cadencemarseille/rust-pcre](https://github.com/cadencemarseille/rust-pcre)
* [erickt/rustpcre](https://github.com/erickt/rustpcre)
* [uasi/rust-pcre](https://github.com/uasi/rust-pcre):  [<img src="https://travis-ci.org/uasi/rust-pcre.png?branch=master">](https://travis-ci.org/uasi/rust-pcre)

## String manipulation
* Slicing w/o copy, stringref
* Ropes
* Tokenizer
* Unicode
  * [Unicode](https://github.com/mozilla/rust/blob/master/src/libextra/unicode.rs)
  * Convertions between text encodings. Ideally, with a customizable way of handling conversion errors.
  * Unicode normalization (NFD, NFC, NFKD, NFKC)
  * Collator (locale sensitive string comparison), with a configurable degree of strictness

## Template engine

* Mustache
  * [erickt/rust-mustache](https://github.com/erickt/rust-mustache): [<img src="https://travis-ci.org/erickt/rust-mustache.png?branch=master">](https://travis-ci.org/erickt/rust-mustache)


## Testing

* QuickCheck
  * [mcandre/rustcheck](https://github.com/mcandre/rustcheck)


## Web Programming

* [skade/widmann](https://github.com/skade/widmann): [<img src="https://travis-ci.org/skade/widmann.png?branch=master">](https://travis-ci.org/skade/widmann)