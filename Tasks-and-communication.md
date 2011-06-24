# Current Status

Task creation and communication works. In master, all tasks are multiplexed onto a single thread and there is no preemption. The compiler may not be enforcing certain invariants though, such as only sending immutable data. Also, not all data types are supported for communication.

On <https://github.com/eholk/rust/commits/threading>, tasks are multiplexed onto multiple OS threads. The number of threads is controlled by the `RUST_THREADS` environment variable. All upcalls are currently serialized on the Big Scheduler Lock. Over time, we can hopefully relax some of this synchronization.

# Design

## Tasks

Tasks are the basic unit of isolation and concurrency in Rust. Tasks do not share memory and communicate only through message passing. By default, tasks are scheduled in an M:N fashion onto operating system threads. The normal mode of operation will be to launch one thread per logical core. The programmer, however, will have the option of assigning certain tasks to a dedicated thread or running the scheduler in 1:1 mode.

When a task is created, a task pointer is returned. This can be used to join a task or kill it. Tasks are implicitly associated with their parent task, and may therefore outlive the function call that spawned them.

## Communication

Communication is accomplished using channels and ports. Each channel is associated with exactly one port, but a port can have many channels associated with it. Channels are the sending end and ports are used for receiving. Only data of a certain type can be sent over each channel and port. 

In order to enforce the no shared mutable state rule, only immutable data can be sent over channels. Likewise, arguments to spawned tasks must also be immutable.

# Historical Design

This section describes more or less how the task and communication system worked in rustboot. Much has changed since then.

Originally, tasks were reference-counted and their lifetime was tied to a local variable. This caused counter-intuitive behavior. For example,

    fn foo() {
        spawn bar();
    }

would immediately kill the newly spawned task. Instead, the usual desired behavior would be written as follows.

    fn foo() {
        let task t = spawn bar();
        join(t);
    }


All of the communication and task primitives were originally core language syntax. Most of these are being moved into the standard library now. This provides more flexibility in many ways.

## Runtime

Overall, the runtime is made up of a `rust_srv`, which owns a `rust_kernel`, which owns one or more `rust_domain`s, each of which own one or more `rust_task`s. Domains corresponded to an operating system thread, and would schedule multiple tasks. Tasks could not migrate between domains. Because only one task would be running at a time in each domain, intra-domain communication happened through directly calling things such as `rust_chan::send`. For communication with tasks in other domains, `rust_proxy` objects are used. These queue messages in a lock-free style on the target domain's message queue, which processes the message at each iteration through the scheduler loop. The kernel's job was to coordinate inter-domain issues.