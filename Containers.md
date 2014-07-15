# Containers in the standard libraries

* Mutable, owned:
    * `std::vec` (dynamic array)
    * `std::str` (string implemented on top of the same dynamic array as `core::vec`)
    * `std::hashmap` (hash table - set and map)
    * `std::trie` (radix trie - set and map, for `uint` keys only at the moment)
    * `extra::treemap` (balanced binary search tree - set and map)
    * `extra::smallintmap` (dense array-based map)
    * `extra::priority_queue` (binary heap)
    * `extra::bitv`
    * `extra::dlist` (doubly-linked list, impl Deque)
    * `extra::ringbuf` (ring buffer, impl Deque)
* Persistent, managed:
    * `extra::list` (singly-linked list)
    * `extra::fun_treemap` (unbalanced binary search tree - not very useful)

# Wanted

* B-tree (Map and Set implementations) - [#4992](https://github.com/mozilla/rust/issues/4992)
* small vector (3-word struct storing small arrays on the stack) - [#4991](https://github.com/mozilla/rust/issues/4991)
* persistent balanced binary search tree (map and set) - [#4987](https://github.com/mozilla/rust/issues/4987)
* persistent heap
* persistent hash-based map/set
* compile-time lookup tables via syntax extensions - [#4864](https://github.com/mozilla/rust/issues/4864)
* multimap

# Issues

* benchmark whether 1.5x is a better growth factor for vectors - [#4961](https://github.com/mozilla/rust/issues/4961)

# Stack/Queue-like containers

## Suggested API convention

* `fn back(&self) -> Option<&self/T>`
* `fn front(&self) -> Option<&self/T>`
* `fn push_back(&mut self, value: T)`
* `fn push_front(&mut self, value: T)`
* `fn pop_back(&mut self) -> Option<T>`
* `fn pop_front(&mut self) -> Option<T>`

This is the same naming convention as C++, and seems to be the most consistent way to do it.

Python uses `append`, `pop`, `appendleft` and `popleft` which isn't as uniform and doesn't lead to an obvious naming convention for equivalents to `back` and `front` (they just use `[0]` and `[-1]`).

## Data structures

* vectors can efficiently implement `front`, `back` `push_back` and `pop_back`
* doubly-linked lists and circular buffers can efficiently implement all of the operations
* a singly-linked list can efficiently implement `front`, `push_front` and `pop_front`, with the addition of `back` and `push_back` if it's either circular or if the container wrapper stores two node pointers
* a priority queue technically doesn't implement any of these operations, it shouldn't share traits

## Traits

This is the tricky part.... there isn't really an obvious way to divide these up into nice traits. A stack could either use `push_front` and `pop_front` or `push_back` and `pop_back`. The same thing applies to a queue, which could either use `push_back` and `pop_front` or `push_front` and `pop_back`.

A `Deque` trait would include all of the operations, so at least that part is easy.