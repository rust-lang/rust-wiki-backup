Notes taken at rust work week, 2011/06/23

## Things that standard libraries might want

* collections
 * list, hash, deque, vec, stack, queue, prioque, trees, set, bitv
 * bitv
 * iteration
* IO
 * AIO, SIO, stdio
 * filesystem
 * path manipulation
 * <> or fileinput
 * timers
* string manipulation
 * slicing w/o copy, stringref
 * regexp
 * ropes
* networking
 * HTTP client / server
* date / time
* math, random
* compression
* libicu
* serialization (protobuf / thrift / json)
* xml
* crypto
* concurrency
 * task management, actor, OTP, [[MapReduce]], pools
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
* DB API
* ZeroMQ
** How will libs like this be integrated with regular nonblocking io?
** ZeroMQ sockets need to be used from a fixed thread, can we do this in rust?
* GUI

## Missing language features
* big
* any
* claim
* note
