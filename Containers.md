Containers in the standard libraries:

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

Obsolete (to be removed):

* `std::dvec` (`~[]` in a mut field) - [#4985](https://github.com/mozilla/rust/issues/4985)
* `std::oldmap` (chaining-based hash table using lots of @ and mut fields) - [#4986](https://github.com/mozilla/rust/issues/4986)
* `std::oldsmallintmap` (like `std::smallintmap`, but with mut fields and a @ box) - [#4738](https://github.com/mozilla/rust/issues/4738)

Wanted:

* radix trie (IntMap, IntSet - perhaps more generic)
* b-tree (either in addition to `std::treemap` or replacing it)
* small vector (3-word struct storing small arrays on the stack)
* LRU cache (doubly-linked list and a hash table pointing at the nodes)
* persistent balanced binary search tree (map and set) - [#4987](https://github.com/mozilla/rust/issues/4987)
* persistent heap
* persistent hash-based map/set
* compile-time lookup tables via syntax extensions - [#4864](https://github.com/mozilla/rust/issues/4864)

# Improvements:

* smallintmap should implement a set type - [#4984](https://github.com/mozilla/rust/issues/4984)
* Add range searches to TreeMap/TreeSet - [#4604](https://github.com/mozilla/rust/issues/4604)
* LinearMap fields should be private - [#4764](https://github.com/mozilla/rust/issues/4764)
* add shrink_to_fit method for vec and hashmap - [#4960](https://github.com/mozilla/rust/issues/4960)