Channel contracts for Rust
==========

This document presents a design for implementing communication using Singularity-style channel contracts for Rust. There are several important features:

* Backwards compatability with the existing communication system.
* A lock-free fast path that allocates no memory and requires only two atomic swaps.
* Slow paths are no more expensive than the current message passing system.
* This entire system can probably be implemented as a syntax extension and some small runtime and libcore changes.

Design
----------

This proposal introduces point to point pipes to serve as the foundation for Rust's communication. A pipe consists of two endpoints, which are not copyable but may transfered between tasks. Each pipe is governed by a contract (or protocol), which specifies what messages are legal in which state. The type system is used to enforce that only legal messages are sent in each state. We can remain compatible with existing Rust message passing code by providing a library of common contracts. As an example, a simple ping-pong protocol would be specified as follows:

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

Protocols are specified from client's perspective. This protocol has two states: `ping` and `pong`. From the `ping` state, the client may send a `ping` message, and the protocol will then transition to the `pong` state. At this point, the client can only receive a `pong` message from the server.

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

The protocol compiler must generate two views of the protocol, one for the client and one for the server. Each protocol has an `init` function, which returns the two endpoints of the appropriate types needed to start the protocol. The protocol compiler would use the above protocol description to generate an implementation similar to the following.

```
mod pingpong {
    fn init() -> (client::ping, server::ping) { ... }
    
    mod client {
        resource ping_(...) { ... };
        type ping = ping_;
        
        resource pong_(...) { ... };
        type pong = pong_;
        
        fn ping(-c: ping) -> pong { ... }
        
        fn pong(-c: pong) -> ping { ... }
    }
    
    mod server {
        resource ping_(...) { ... };
        type ping = ping_;
        
        resource pong_(...) { ... };
        type pong = pong_;
        
        fn ping(-c: ping) -> (pong, ()) { ... }
        
        fn pong(-c: pong) -> ping { ... }       
    }
}
```

Note that each operation on an endpoint consumes the endpoint and returns a new one. This is how Rust's type system is used to enforce that only legal messages are sent at any given time.

The actual concrete types for each of the states have some flexibility. The important part is that they be sendable but noncopyable. Protocols that allow for multiple messages, and different payloads will be more complex, but these can be handled by using an enum describing the legal messages.

# Select #

For simplicity, we'll only consider `select2` for now. Select is harder under this system due to the linear nature of pipe endpoints. We can special case select for small numbers, but a fully general select will probably require a macro at least. `select2` will be written something like this:

```
fn select2<P1, P1_, M1, P2, P2_, M2>(
    -pipe1: P1, trans1: fn(-P1) -> (P1_, M1),
    -pipe2: P2, trans2: fn(-P2) -> (P2_, M2)) 
    -> either<(P1_, P2, M1), (P1, P2_, M2)> { ... }
```

Select must consume both of its endpoints, and it will return the next endpoint for the pipe that yielded data, and the old endpoint unchanged for the pipe that did not receive data.

Implementation
----------

Pipes are represented in memory as a status field and a payload. The payload is basically an enum of all the legal messages in the current state. It is initial empty (that is, full of invalid data), but it will be filled in by the sender. The status field is used to synchronize between senders and receivers.

The status field indicates one of the following states:

* `empty` - There is no data in the channel yet.
* `full` - There is valid data in the channel.
* `blocked(task)` - There is a task waiting on data from this channel. `task` is a pointer or other identifier for the task that is blocked.
* `terminated` - One endpoint has been destroyed.

Ideally, these four states (including the task identifier/pointer) could be packed into a single word-sized value.

Channels initially start out in the `empty` state, and states evolve based on various channel operations.

# Channel Operations #

This section provides algorithms for how to send to, receive from, and terminate a pipe endpoint.

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

If the task ended up going to sleep, when the scheduler wakes the task again, we know that either there is data available, or the endpoint has been terminated. Upon being woken up, the task should check the status field. It is either `terminated` or `full`. If `terminated`, the receiver simply tears down the channel and informs the caller that the sender closed the endpoint. Otherwise, the receiver tears down the endpoint and returns the message payload to the caller.

## Terminate (Sender side) ##

The sender may instead destroy its channel sending any data. To do this, it swaps `terminated` into the channel's status field. The sender now takes the following action based on the previous state:

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

Select is a somewhat tricky synchronization problem that will require some changes to the scheduler. The basic idea is that a receiver will first swap itself as blocked in the status fields for each of the endpoints it wishes to select on. If any of these were full, it can simply cancel the select and return the pre-existing data. If it gets through all of the ports and finds no data, then it tells the scheduler to put it to sleep.

Once one of the senders wakes up the task, the receiver will go back through all of the endpoints it is blocked on and first swap in `empty` as the status. If the previous status was `full`, then it will set the status back to `full` so that future receives on that endpoint will work correctly.

There are a few changes to the scheduler that are necessary to make this safe. Currently, tasks block on only one port (or port group), so it always knows why it was woken up. We need to reverse this relationship. Instead, the task is woken up and it is provided with the ID of the endpoint or possibly other entity that woke it up.

More importantly, we need to make sure that sends that happen while the task is setting up a select operation still work correctly. To do this, the scheduler will gain a new `block_pending` state. Tasks set this to indicate to the scheduler that they might block soon, but need to do some more processing. Thus, the first thing a task does when setting up a select is to set its state to `block_pending`. If any data is already available, the task sets its state back to `running` before returning. Otherwise, it goes all the way to sleep, and its state becomes `blocked`.

The wakeup path changes slightly as well. If the target was already in the `running` state, wakeup is just a no-op. If it is `block_pending` or `blocked`, the target transitions back into `running`. However, when a task goes to transition itself from `block_pending` to `blocked`, if another task has already transitioned it back to `running`, it simply behaves as if it has already been woken up. Waking up a task in the `block_pending` state simply cancels the block.

# Integration with Rust #

They are a couple of strategies for how to integrate this system into the Rust compiler. The basic idea is that we need way to take a channel contract and compile it into a pair of modules that encode the protocols into the type system.

The least risky approach is to build a tool that works sort of like an RPC compiler. You specify a channel contract in a separate file, and then run it through a contract compiler, which will generate an appropraite `.rs` file.

Alternately, we could integrate this into the compiler. A good way to do this is to make a syntax extension, or perhaps a macro if our macro system is powerful enough. The other option is to integrate it into Rust. This approach will involve adding a lot of typechecking and trans code, so this would not be my first choice.

It makes sense to first build this as a message passing system that runs alongside the existing one. Once it has proven itself, we have the option of trying to reimplement the existing message passing system in terms of channel contracts. The following contract would give us a starting point for implementing ports and chans like we have now.

```
#proto portchan<T: send> {
    portchan {
        send data(T) -> portchan;
    }
}
```

This contract could be wrapped with some other data structures and functions to provide pretty much the same interface as the existing ports and channels.

This system will require some runtime support in order to handle synchronization where the scheduler must be involved (that is, on the slow paths).

Optimizations
----------

In the most general case, the sender is always responsible for allocating space for the next message in the protocol. This means that every send will incur a memory allocation. There are several things we can do to avoid much of this cost.

The first thing is to pre-allocate space for several messages, so that allocation will be more infrequent. This makes memory accounting a little trickier, but not insurmountable.

For many protocols, however, it is possible to statically determine the maximum amount of buffer space that will ever be needed. In this case, we can simply allocate all of this space up front, and then sending never incurs an allocation cost. The key characteristic is whether a protocol is bounded. Bounded protocols are ones in which every loop in the state machine has both a send operation and a receive operation. This property puts a bound on how much data can ever be in flight without the receiver acknowledging it. The `pingpong` protocol above is an example of a bounded protocol. On the other hand, the `portchan` protocol is unbounded.

Conclusion
----------

This describes a proposal for adding pipes to Rust. Pipes are communication pathways with two endpoints, governed by a communication protocol that enforces when what messages can travel in which direction. The communication protocol (or contract) is encoded into Rust's type system, which provides strong compile-time correctness guarantees. The system provides the following advantages.

* A fast path that involves no locks on either the send or receive side.
* Easy detection of either endpoint closing the pipe.
* The single-use nature of endpoints makes the synchronization protocols easier to reason about and get correct.
* Bounded protocols can preallocate all buffer space and thus incur no allocation cost for sending messages.
* Many existing communication patterns can be described in terms of pipes and protocols.
* This proposal requires few language changes. It can be implemented using a syntax extension and some library/runtime changes.

There are, however, some shortcomings.

* The added safety and performance potential imposes a higher annotation burden on the programmer. This can be mitigated by providing a library of standard protocols, such as traditional port and chan system.
* The many to one nature of today's channels and ports isn't really possible, as pipe endpoints are noncopyable. Something like copy constructors could help us get around this, but it will not solve all the problems.

This system provides important performance and safety improvements, and yet will likely be able to express many of the patterns that occur in practice. This system is very similar to the one that was used in the Singularity operating system, and this provides evidence that it could also serve Rust's needs.

Feedback
----------

* pcwalton pointed out that this would make it easy to do send-and-yield-to-target, since the status field includes the target. Since the sender has to preallocate response space, it could go ahead and mark itself as blocked on the response, and thus the ping-pong could be super fast. We could make `sendrecv` a primitive.
* graydon
  * says this looks good for what it's designed for, and likes how it teases out protocol boundedness as an optimization criterion.
  * suggests talking to the XPIDL people, and maybe making a promela (or similar) model to help track down corner cases.
  * likes that I handled ownership of data in flight, and is happy if I keep experimenting with this. He suggests a syntax extension.
  * how to do many-to-one? Maybe with exclusive arc? 
  * for N:1, make sure the actions of each of the N don't interfere besides contending for attention of the one.
  * bounded buffer size ≅ bounded number of clients
* From talking with Graydon, I realized selecting among many pipes with uniform types is really easy. This simplifies N:1 or 1:M a lot, I would guess.
