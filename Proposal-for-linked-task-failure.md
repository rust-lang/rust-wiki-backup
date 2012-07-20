# Linked task failure

Linked task failure supports configuring a program's tasks such that when one invokes `fail`, it can automatically kill a certain set of other tasks.

## Linked failure modes

The way tasks will propagate failures is configured at spawn-time. There are three supported spawn modes:

- Bidirectional (linked) failure - Either the child or the parent can kill the other in the event of failure.
- Unlinked failure (no propagation) - Failure by either the child or the parent task will not affect the other.
- Unidirectional (parented) failure - If the parent fails, the child will die, but if the child fails, the parent will not.

The interfaces for each will be approximately:

    do task::task().linked().spawn { ... }   // default - equivalent to task::spawn
    do task::task().unlinked().spawn { ... }
    do task::task().parented().spawn { ... } // equivalent to unlinked().parented()

The method-chaining interface is based around a builder that manages spawn options. It can also be used to set other task configuration options, such as `notify_port` and `add_wrapper`. Because these are noncopyable, attempting to duplicate a builder in the middle of the configuration process will fail.

## Taskgroups

The use of `spawn_linked` gives rise to groups of tasks in which if one fails, all will be killed. I call these taskgroups.

In addition to the three spawn modes, there will be a special interface to explicitly spawn into an existing taskgroup:

    let g = task::group();
    do task::task().in_group(g).spawn { ... }
    do task::task().in_group(g).spawn { ... }

## Taskgroup tree and failure propagation

Spawning parented tasks results in a task tree much like that of the UNIX process model. Failure propagates downwards; if task A parents B which parents C, and A fails, both B and C will be killed.

It is more useful to think of the basic unit of the task hierarchy as taskgroups instead of tasks, because failure always propagates through taskgroups. For example, here, the innermost task will be killed.

    do task::task().parented().spawn {
        do task::spawn { // linked
            do task::task().parented().spawn {
                loop { task::yield(); }
            }
        }
    }
    fail;

Another interaction arises from being able to spawn into other groups. In this example, the first grandchild must be killed, even though if the middle task exits first there was a period of time in which the middle taskgroup had no tasks.

    let g = task::group();
    do task::task().parented().in_group(g).spawn {
        do task::task().parented().spawn {
            loop { task::yield(); }
        }
    }
    do task::task().parented().in_group(g).spawn {
        fail;
    }

## Implementation

Each task keeps a taskgroup data structure in task-local storage. These each have references to exclusive ARCs, one per taskgroup, which contain a list of all tasks in the group. When a task exits cleanly, it removes itself from the shared list. When a task fails, it iterates over the shared list and kills each task on the list. (A task's presence on the list indicates that it's still alive, and safe to be killed.)

### Current

I use dvec for the lists, because it's the best sendable data structure we have (see issue #2816). Tasks know their own index into the list. By keeping an additional list of empty slots in the dvec, both addition and removal become O(1).

    type taskgroup_arc = exclusive<{
        refcount:             uint,
        tasks:                dvec<option<rust_task *>>,
        tasks_empty_slots:    dvec<uint>,
        children:             dvec<option<rust_task *>>,
        children_empty_slots: dvec<uint>,
    }>;
    class taskgroup {
        me:        *rust_task,
        tasks:     taskgroup_arc,
        my_pos:    uint,
        ancestors: dvec<{parent: taskgroup_arc, my_pos: uint}>;
        is_main:   bool
    }

Key points:

- When a taskgroup arc's tasks.len() == tasks_empty_slots.len(), it means all tasks in it are gone and the dvec can be recycled (resized back to 0).
- The refcount in the arc is different from the arc's built-in refcount. It indicates the possibility for tasks to ever exist in this group which might be able to kill us. It might exceed the number of tasks in the group if the explicit `task::group()` interface is used. When tasks.len() == tasks_empty_slots.len(), the group is 'dormant'. When refcount == 0, the group is 'dead', and descendant taskgroups may then stop accounting for it.

#### Space/time complexity quirks

Because each task has a reference to each of its ancestors, there is potential for massive space buildup in the case of a 'spawn ladder', like so:

    fn child_no(generation: uint) {
        if generation < BIGNUM {
            do task::task().parented().spawn {
                child_no(generation-1);
            }
        }
    }

There may be massive buildup in the size of the ancestor list: it would be O(n) in the number of generations that ever existed, not just O(n) in the number of live generations. We could somewhat alleviate this by having tasks GC 'dead' groups from their ancestor list right before spawning a child. Then the maximum size of the ancestor list would be O(max # simultaneous live generations).

Another problem: if O(n) intermediate generations are alive and have ancestor lists O(n) long, suddenly this is O(n^2) metadata for O(n) tasks.

### Ideal implementation

The reason/blocker for the current implementation is the need for each task to know its index in each ancestor's list, so it can remove itself in constant time. If we had sendable hashtables, we could solve the O(n^2) problem with:

    type taskgroup_arc = exclusive<{
        refcount:             uint,
        tasks:                hashset<rust_task *>,
        children:             hashset<rust_task *>,
    }>;
    enum ancestors {
        nil,
        cons(exclusive<{
            parent_group: taskgroup_arc,
            ancestors:    ancestors
        }>)
    }
    class taskgroup {
        me:        *rust_task,
        tasks:     taskgroup_arc,
        my_pos:    uint,
        ancestors: ancestors,
        is_main:   bool
    }
