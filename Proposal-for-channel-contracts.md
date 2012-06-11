Channel contracts for Rust
==========

This document presents a design for implementing communication using Singularity-style channel contracts for Rust. There are several important features:

* Backwards compatability with the existing communication system.
* A lock-free fast path that allocates no memory and requires only two atomic swaps.
* Slow paths are no more expensive than the current message passing system.
* No additional language support is required, other than a syntax extension.

Design
----------

Here I need to answer these questions:

1. What are channels?
2. What are channel contracts? How do I specify them?
3. How does select work? What is its type?
4. How do I do patterns like 1:1, N:1, 1:M, N:M?

This proposal introduces point to point channels as the fundamental building block for Rust's communication. A channel consists of two endpoints, which are not copyable but may transfered between tasks. Each channel is governed by a contract, which specifies which messages are legal in which state. The type system is used to enforce that only legal messages are sent in each state. We can remain compatible with existing Rust message passing code by providing a library of common contracts. A simple ping-pong protocol would be specified as follows:

```
#proto pingpong {
    ping {
        send ping() -> pong;
    }
    
    pong {
        recv pong() -> ping;
    }
}
```

Protocols (or contracts) are specified from client's perspective. This protocol has two states: `ping` and `pong`. From the `ping` state, the client may send a `ping` message, and the protocol will then transition to the `pong` state. At this point, the client can only receive a `pong` message from the server.

This syntax extension would go through a protocol compiler which basically translates states into noncopyable, sendable types and messages into functions that consume a state and any message payloads and return a new state. The protocol can be used as follows.

```
fn client(chan: pingpong::client::ping) {
    let chan = pingpong::client::ping(chan);
    let (chan, data) = pingpong::client::pong(chan);
}

fn server(chan: pingpong::server::pong) {
    let (chan, data) = pingpong::server::ping(chan);
    let chan = pingpong::server::pong(chan);
}

fn main() {
    let (client, server) = pingpong::init();
    
    client(client);
    server(server);
}
```

The protocol compiler must generate two views of the protocol, one for the client and one for the server. Each protocol has an `init` function, which returns the two endpoints of the appropriate types needed to start the protocol.

* Select

Implementation
----------

This section will basically describe the code the channel compiler produces, for the naive cases.

* Channel representation
* Channel states
  * Empty
  * Blocked
  * Full
  * Terminated
* Channel states impose a cleanup discipline, since we can detect when the other side has closed the connection.

Optimizations
----------

This describes what the compiler can do in the case of bounded protocols, and also using queues to amortize or avoid the cost of memory allocation.

Backwards Compatibility
----------

Conclusion
----------
