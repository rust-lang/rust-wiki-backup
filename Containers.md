# Containers in the standard libraries

* Mutable:
    * `core::dlist` (doubly-linked list - uses @ but avoiding that would require `unsafe`)
    * `core::vec` (dynamic array)
    * `core::str` (string implemented on top of the same dynamic array as `core::vec`)
    * `core::hashmap` (hash table - set and map)
    * `std::treemap` (balanced binary search tree - set and map)
    * `std::smallintmap` (dense array-based map)
    * `std::deque` (ring buffer)
    * `std::priority_queue` (binary heap)
    * `std::bitv`
* Persistent:
    * `std::list` (singly-linked list)
    * `std::fun_treemap` (unbalanced binary search tree - not very useful)

# Obsolete (to be removed)

* `std::dvec` (`~[]` in a mut field) - [#4985](https://github.com/mozilla/rust/issues/4985)
* `std::oldmap` (chaining-based hash table using lots of @ and mut fields) - [#4986](https://github.com/mozilla/rust/issues/4986)

# Wanted

* radix trie (IntMap, IntSet - perhaps more generic)
* B-tree (Map and Set implementations) - [#4992](https://github.com/mozilla/rust/issues/4992)
* small vector (3-word struct storing small arrays on the stack) - [#4991](https://github.com/mozilla/rust/issues/4991)
* LRU cache (doubly-linked list with a hash table pointing at the nodes) - [#4988](https://github.com/mozilla/rust/issues/4988)
* persistent balanced binary search tree (map and set) - [#4987](https://github.com/mozilla/rust/issues/4987)
* persistent heap
* persistent hash-based map/set
* compile-time lookup tables via syntax extensions - [#4864](https://github.com/mozilla/rust/issues/4864)

# Issues

* smallintmap should implement a set type - [#4984](https://github.com/mozilla/rust/issues/4984)
* Add range searches to TreeMap/TreeSet - [#4604](https://github.com/mozilla/rust/issues/4604)
* LinearMap fields should be private - [#4764](https://github.com/mozilla/rust/issues/4764)
* add shrink_to_fit method for vec and hashmap - [#4960](https://github.com/mozilla/rust/issues/4960)
* split the Map and Set traits into Map, MutableMap, PersistentMap and Set, MutableSet, PersistentSet - [#4989](https://github.com/mozilla/rust/issues/4989)
* Move dlist to std - [#3549](https://github.com/mozilla/rust/issues/3549)
* benchmark whether 1.5x is a better growth factor for vectors - [#4961](https://github.com/mozilla/rust/issues/4961)
* add reserve/reserve_at_least to std::deque - [#4994](https://github.com/mozilla/rust/issues/4994)

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

This is the tricky part.... there isn't really an obvious way to divide these up into nice traits. A stack could either use `push_front` and `pop_front` or `push_back` and `pop_back`. The same thing applies to a queue, which could either use `push_back` and `pop_front` or `push_front and `pop_back`.

A `Deque` trait would include all of the operations, so at least that part is easy.