Notes taken at rust work week, 2011/06/23

## Things that standard libraries might want

* collections
 * list, hash, deque, vec, stack, queue, prioque, trees, set, bitv
 * bitv
 * iteration
 * pure collections
* IO
 * AIO, SIO, stdio
 * filesystem
 * path manipulation
 * <> or fileinput
 * timers
* string manipulation
 * slicing w/o copy, stringref
 * [regexp](https://github.com/elly/rustpcre)
 * [ropes](https://github.com/mozilla/rust/blob/master/src/libstd/rope.rs)
* networking
 * HTTP client / server
 * [UUID](https://github.com/erickt/rust-uuid), GUID
* date / time
* math, random
* [compression](https://github.com/elly/rustzlib)
* [libicu](https://github.com/mozilla/rust/blob/master/src/libstd/unicode.rs)
* serialization
 * protobuf
 * thrift
 * xml
 * [json](https://github.com/mozilla/rust/blob/master/src/libstd/json.rs)
 * [tnetstring](https://github.com/erickt/rust-tnetstring)
* [crypto](https://github.com/elly/rustcrypto)
* concurrency
 * task management, actor, OTP, [[Bikeshed mapreduce]], pools
* low-level OS services
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
  * mongodb
  * redis
  * [sqlite](https://github.com/linuxfood/rustsqlite)
* [ZeroMQ](https://github.com/erickt/rust-zmq)
 * How will libs like this be integrated with regular nonblocking io?
 * ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?
* GUI

## Missing language features
* big
* any
* claim
* note
