Channel contracts for Rust
==========

*** This is still a work in progress ***

This document presents a design for implementing communication using Singularity-style channel contracts for Rust. There are several important features:

* Backwards compatability with the existing communication system.
* A lock-free fast path that allocates no memory and requires only two atomic swaps.
* Slow paths are no more expensive than the current message passing system.
* This entire system can probably be implemented as a syntax extension.

Design
----------

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

# Select, etc #

Still to come...

Here I need to answer these questions:

3. How does select work? What is its type?
4. How do I do patterns like 1:1, N:1, 1:M, N:M?

* Select - working out the types is kind of tricky, since one endpoint will be consumed, but we don't know which. It will probably have to give you your data, as well as a vector of endpoints with the one that received replaced.

Implementation
----------

Channels will be represented in memory as a status field and a payload. The payload is basically an enum of all the legal messages in the current state. It is initial empty (that is, full of invalid data), but it will be filled in by the sender. The status field is used to synchronize between senders and receivers.

The status field indicates one of the following states:

* `empty` - There is no data in the channel yet.
* `full` - There is valid data in the channel.
* `blocked(task)` - There is a task waiting on data from this channel. `task` is a pointer or other identifier for the task that is blocked.
* `terminated` - One endpoint has been destroyed.

Ideally, these four states (including the task identifier/pointer) could be packed into a single word-sized value.

Channels initially start out as empty. Now we discuss the various channel operations.

# Channel Operations #

## Send ##

1. Place the message contents in the payload section. This may involve allocating space for and storing a pointer to the next stage in the protocol, which gets returned by send.
2. Atomically swap `full` into the status field. Now look at the old value:
   1. If it was `empty`, no further work is needed.
   2. If it was `full`, something went wrong. Fail.
   3. If it was `blocked(task)`, there is a task waiting on this data. Tell the scheduler to wake up the task.
   4. If it was `terminated`, the receiver has destroyed its endpoint and thus the message will never be received. The sender now takes responsibility for running destructors on the data in the channel.

At this point, the sender relinquishes ownership of the channel and no longer bears any cleanup responsibility.

## Receive ##

1. Atomically swap `blocked(task)` into the status field, where `task` points to the currently running task.
2. Check the old value.
   1. If it was `empty`, tell the scheduler to put us to sleep.
   2. If it was `full`, return the contents of the payload field.
   3. If it was `blocked(task)`, something has gone wrong. Fail.
   4. If it was `terminated`, the sender has destroyed its endpoint. Thus, no data will ever arrive. Return, informing the caller that the sender has closed the endpoint.

If the task ended up going to sleep, when the scheduler wakes the task again, we know that either there is data available, or the endpoint has been shutdown. Upon being woken up, the task should check the status field. It is either `terminated` or `full`. If `terminated`, the receiver simply tears down the channel and informs the caller that the sender closed the endpoint. Otherwise, the receiver tears down the endpoint and returns the message payload to the caller.

## Terminate (Sender side) ##

The sender may instead destroy its channel without shutting it down. To do this, it swaps `terminated` into the channel's status field. The sender now takes the following action based on the previous state:

* `empty` - The receiver hasn't run yet, so do nothing. The receiver now has cleanup responsibility.
* `full` - This shouldn't happen. Fail.
* `blocked(task)` - A task is waiting on data. Tell the scheduler to wake up this task and it will handle the remainder of the cleanup.
* `terminated` - The receiver is not going to receive. Go ahead and free the channel memory. The payload contents are invalid in this state, so ignore them.

## Terminate (Receiver side) ##

If the receiver wishes to destroy a channel without receiving, it simply swaps the `terminated` state into the status field. Based on the old state, the receiver takes the following actions.

* `empty` - The sender endpoint is still live, so the sender will handle cleanup.
* `full` - The sender has shut down its endpoint, and there a message. Cleanup the message payload and the rest of the channel structure.
* `blocked(task)` - This is an illegal state. Fail.
* `terminated` - The sender has terminated its endpoint. There is no data in the payload, so simply free the channel's memory.

# Implementing select #

Still to come...


Optimizations
----------

Still to come...

This describes what the compiler can do in the case of bounded protocols, and also using queues to amortize or avoid the cost of memory allocation.

* Pre-allocating buffers for bounded protocols
* Allocating buffers in chunks for unbounded protocols, reusing chunks to amortize or completely remove `malloc` latency

Backwards Compatibility
----------

Still to come...

* Show a contract for ports/channels.

Conclusion
----------

Still to come...

* Strengths and weaknesses
  * Non-blocking fast path
  * Not zero-copy. It looks like at least two copies in general. We might be able to get down to one copy using region pointers. For types that move by-pointer, we effectively get zero-copy.
