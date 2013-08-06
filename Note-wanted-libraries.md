Notes taken at rust work week, 2011/06/23

## Things that standard libraries might want

* collections (see [[Containers]])
* IO
 * AIO, SIO, stdio
 * [filesystem](https://github.com/mozilla/rust/blob/master/src/libstd/os.rs)
 * [path manipulation](https://github.com/mozilla/rust/blob/master/src/libstd/path.rs)
 * [<> or fileinput](https://github.com/mozilla/rust/blob/incoming/src/libextra/fileinput.rs)
 * timers
* string manipulation
 * slicing w/o copy, stringref
 * [regexp](https://github.com/elly/rustpcre) (external)
 * [ropes](https://github.com/mozilla/rust/blob/master/src/libextra/rope.rs)
 * simple tokenizer
* Localizability
 * one aspect of L10n is to map a key to a text, based on the current locale (eg Java's [ResourceBundle](http://docs.oracle.com/javase/7/docs/api/java/util/ResourceBundle.html) or [GNU gettext](http://www.gnu.org/software/gettext/))
 * another aspect is to format a string based on the current locale (eg Java's [MessageFormat](http://docs.oracle.com/javase/7/docs/api/java/text/MessageFormat.html))
 * See [issue #4630](https://github.com/mozilla/rust/issues/4630)
* networking
  * HTTP
    * client
      * [rust-http-client](https://github.com/mozilla-servo/rust-http-client) (external)
    * server
      * [rusthttpserver](http://hg.chrismorgan.info/rusthttpserver)
      * [rwebserve](https://github.com/jesse99/rwebserve) (external)
 * [URI/URL](https://github.com/mozilla/rust/blob/master/src/libextra/net_url.rs)
 * [UUID](https://github.com/erickt/rust-uuid) (external)
 * GUID
* date and time
  * [rust_datetime](https://github.com/tedhorst/rust_datetime) (external)
* math
  * [lmath](https://github.com/bjz/lmath-rs) (external)
  * [rusty-math](https://github.com/z0w0/rusty-math) (external)
* [random](https://github.com/mozilla/rust/blob/master/src/libcstd/rand.rs)
* [compression](https://github.com/elly/rustzlib) (external)
* [libicu](https://github.com/mozilla/rust/blob/master/src/libextra/unicode.rs)
 * Convertions between text encodings. Ideally, with a customizable way of handling conversion errors.
 * Unicode normalization (NFD, NFC, NFKD, NFKC)
 * Collator (locale sensitive string comparison), with a configurable degree of strictness
* serialization/encoding
 * [base64](https://github.com/mozilla/rust/blob/master/src/libextra/base64.rs)
 * [CSV](https://github.com/grahame/rust-csv) (external)
 * [json](https://github.com/mozilla/rust/blob/master/src/libextra/json.rs)
 * protobuf
 * thrift
 * [Cap'n Proto](https://github.com/dwrensha/capnproto-rust) (external)
 * [tnetstring](https://github.com/erickt/rust-tnetstring) (external)
 * xml
* file formats
 * zip file format
 * tar file format
* [crypto](https://github.com/elly/rustcrypto) (external)
* concurrency
 * task management, actor, OTP, [[Bikeshed mapreduce]], pools
* low-level OS services
* Simple search on a filesystem (eg Ruby's [glob](http://ruby-doc.org/core-2.0/Dir.html#method-c-glob))
* unit testing
* FFI, ctypes
* dlopen, os proceses
* standard predicates
 * text, numeric, sorted
* error-trapping wrappers, in-place task?
 * Consistent error handling
* quotas, accounting
* reflection

## Things that do not belong in std
* DB API (sql, nosql)
  * postgres
  * [mongodb](https://github.com/10gen-interns/mongo-rust-driver-prototype) (external)
  * [redis](https://github.com/mneumann/rust-redis) (external)
  * [sqlite](https://github.com/linuxfood/rustsqlite) (external)
* [ZeroMQ](https://github.com/erickt/rust-zmq) (external)
 * How will libs like this be integrated with regular nonblocking io?
 * ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?
* GUI
 * [Cocoa](https://github.com/pcwalton/rust-cocoa) (external)

## Missing language features
* big
* any
* claim
* note