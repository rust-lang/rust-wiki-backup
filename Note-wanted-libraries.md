[Travis CI Status](http://hiho.io/rust-ci/)

## Collections

See [[Containers]].

## Audio

See [[Computer Graphics and Game Development]].

## Compression
 * zip file format
 * tar file format
 * [rust-snappy](https://github.com/thestinger/rust-snappy) (external)
 * [rustzlib](https://github.com/elly/rustzlib) (external)
 
## Concurrency
 * task management, actor, OTP, [[Bikeshed mapreduce]], pools
 
## Crypto

Do not belong in std.

* [crypto](https://github.com/mozilla/rust/tree/master/src/libextra/crypto)
* [rustcrypto](https://github.com/erickt/rustcrypto) (external)
  
## Database Access

Do not belong in std.

* NoSql
  * [leveldb](https://github.com/lht/rust-leveldb) (external)
  * [mongodb](https://github.com/10gen-interns/mongo-rust-driver-prototype) (external)
  * [redis](https://github.com/mneumann/rust-redis) (external)
* SQL
  * [postgres](https://github.com/sfackler/rust-postgres) (external)
  * [sqlite](https://github.com/linuxfood/rustsqlite) (external)
 
## Date and Time
  * [rust_datetime](https://github.com/tedhorst/rust_datetime) (external)
  * [rust-datetime](https://github.com/luisbg/rust-datetime) (external)
  
## Encoding
 * [Base64](https://github.com/mozilla/rust/blob/master/src/libextra/base64.rs)
 * [Cap'n Proto](https://github.com/dwrensha/capnproto-rust) (external)
 * [CSV](https://github.com/grahame/rust-csv) (external)
 * [HTML](https://github.com/veddan/rust-htmlescape) (external)
 * [JSON](https://github.com/mozilla/rust/blob/master/src/libextra/json.rs)
 * [ProtocolBuffers](https://github.com/stepancheg/rust-protobuf) (external)
 * Thrift
 * [tnetstring](https://github.com/erickt/rust-tnetstring) (external)
 * XML
      * [sax-rs](https://github.com/bjz/sax-rs) (external)

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
  * [lmath](https://github.com/bjz/lmath-rs) (external)
  * [rusty-math](https://github.com/z0w0/rusty-math) (external)

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
* [HTTP](https://github.com/chris-morgan/rust-http) (external)
* [URI/URL](https://github.com/mozilla/rust/blob/master/src/libextra/url.rs)
* [UUID](https://github.com/mozilla/rust/blob/master/src/libextra/uuid.rs)
* GUID
* [ZeroMQ](https://github.com/erickt/rust-zmq) (external)
  * How will libs like this be integrated with regular nonblocking io?
  * ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?
 
## Random Numbers
* [random](https://github.com/mozilla/rust/blob/master/src/libcstd/rand.rs)

## Regular Expressions
See https://github.com/mozilla/rust/issues/3591
* [rust-pcre](https://github.com/uasi/rust-pcre) (external)
* [rustpcre](https://github.com/erickt/rustpcre) (external)

## String manipulation
* Slicing w/o copy, stringref
* Ropes
* Tokenizer
* Unicode
  * [Unicode](https://github.com/mozilla/rust/blob/master/src/libextra/unicode.rs)
  * Convertions between text encodings. Ideally, with a customizable way of handling conversion errors.
  * Unicode normalization (NFD, NFC, NFKD, NFKC)
  * Collator (locale sensitive string comparison), with a configurable degree of strictness