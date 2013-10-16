[Travis CI Status](http://hiho.io/rust-ci/)

## Things that standard libraries might want

* collections (see [[Containers]])
* compression
 * zip file format
 * tar file format
 * [rust-snappy](https://github.com/thestinger/rust-snappy) (external)
 * [rustzlib](https://github.com/elly/rustzlib) (external)
* concurrency
 * task management, actor, OTP, [[Bikeshed mapreduce]], pools
* date and time
  * [rust_datetime](https://github.com/tedhorst/rust_datetime) (external)
  * [rust-datetime](https://github.com/luisbg/rust-datetime) (external)
* encoding
 * [Base64](https://github.com/mozilla/rust/blob/master/src/libextra/base64.rs)
 * [Cap'n Proto](https://github.com/dwrensha/capnproto-rust) (external)
 * [CSV](https://github.com/grahame/rust-csv) (external)
 * [HTML](https://github.com/veddan/rust-htmlescape) (external)
 * [JSON](https://github.com/mozilla/rust/blob/master/src/libextra/json.rs)
 * [ProtocolBuffers](https://github.com/stepancheg/rust-protobuf) (external)
 * Thrift
 * [tnetstring](https://github.com/erickt/rust-tnetstring) (external)
 * xml
      * [sax-rs](https://github.com/bjz/sax-rs) (external)
* IO
 * AIO, SIO, stdio
 * [filesystem](https://github.com/mozilla/rust/blob/master/src/libstd/os.rs)
 * [path manipulation](https://github.com/mozilla/rust/blob/master/src/libstd/path.rs)
 * [<> or fileinput](https://github.com/mozilla/rust/blob/incoming/src/libextra/fileinput.rs)
 * timers
* string manipulation
 * slicing w/o copy, stringref
 * [regexp](https://github.com/elly/rustpcre) (external)
 * ropes
 * simple tokenizer
* Localizability
 * one aspect of L10n is to map a key to a text, based on the current locale (eg Java's [ResourceBundle](http://docs.oracle.com/javase/7/docs/api/java/util/ResourceBundle.html) or [GNU gettext](http://www.gnu.org/software/gettext/))
 * another aspect is to format a string based on the current locale (eg Java's [MessageFormat](http://docs.oracle.com/javase/7/docs/api/java/text/MessageFormat.html))
 * See [issue #4630](https://github.com/mozilla/rust/issues/4630)
* math
  * [lmath](https://github.com/bjz/lmath-rs) (external)
  * [rusty-math](https://github.com/z0w0/rusty-math) (external)
* networking
 * [HTTP](https://github.com/chris-morgan/rust-http) (external)
 * [URI/URL](https://github.com/mozilla/rust/blob/master/src/libextra/url.rs)
 * [UUID](https://github.com/mozilla/rust/blob/master/src/libextra/uuid.rs)
 * GUID
* [random](https://github.com/mozilla/rust/blob/master/src/libcstd/rand.rs)
* [Unicode](https://github.com/mozilla/rust/blob/master/src/libextra/unicode.rs)
 * Convertions between text encodings. Ideally, with a customizable way of handling conversion errors.
 * Unicode normalization (NFD, NFC, NFKD, NFKC)
 * Collator (locale sensitive string comparison), with a configurable degree of strictness
* low-level OS services
* Simple search on a filesystem (eg Ruby's [glob](http://ruby-doc.org/core-2.0/Dir.html#method-c-glob))
* unit testing
* FFI, ctypes
* dlopen, os processes
* standard predicates
 * text, numeric, sorted
* error-trapping wrappers, in-place task?
 * Consistent error handling
* reflection

## Things that do not belong in std

* Crypto
  * [crypto](https://github.com/mozilla/rust/tree/master/src/libextra/crypto)
  * [rustcrypto](https://github.com/erickt/rustcrypto) (external)
* Database access
  * SQL
      * [postgres](https://github.com/sfackler/rust-postgres) (external)
      * [sqlite](https://github.com/linuxfood/rustsqlite) (external)
  * NoSql
      * [leveldb](https://github.com/lht/rust-leveldb) (external)
      * [mongodb](https://github.com/10gen-interns/mongo-rust-driver-prototype) (external)
      * [redis](https://github.com/mneumann/rust-redis) (external)
* GUI
 * [rust-cocoa](https://github.com/mozilla-servo/rust-cocoa) (external)
 * [wxRust](https://github.com/kenz-gelsoft/wxRust) (external)
* quotas, accounting
* [ZeroMQ](https://github.com/erickt/rust-zmq) (external)
 * How will libs like this be integrated with regular nonblocking io?
 * ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?

## Missing language features
* big
* any
* claim
* note