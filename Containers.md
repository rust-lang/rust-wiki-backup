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
* B-tree (either in addition to `std::treemap` or replacing it)
* small vector (3-word struct storing small arrays on the stack) - [#4991](https://github.com/mozilla/rust/issues/4991)
* LRU cache (doubly-linked list with a hash table pointing at the nodes) - [#4988](https://github.com/mozilla/rust/issues/4988)
* persistent balanced binary search tree (map and set) - [#4987](https://github.com/mozilla/rust/issues/4987)
* persistent heap
* persistent hash-based map/set
* compile-time lookup tables via syntax extensions - [#4864](https://github.com/mozilla/rust/issues/4864)

# Issues:

* smallintmap should implement a set type - [#4984](https://github.com/mozilla/rust/issues/4984)
* Add range searches to TreeMap/TreeSet - [#4604](https://github.com/mozilla/rust/issues/4604)
* LinearMap fields should be private - [#4764](https://github.com/mozilla/rust/issues/4764)
* add shrink_to_fit method for vec and hashmap - [#4960](https://github.com/mozilla/rust/issues/4960)
* Do something about std::deque - [#2343](https://github.com/mozilla/rust/issues/2343)
* Deque shouldn't require copyable types - [#3748](https://github.com/mozilla/rust/issues/3748)
* split the Map and Set traits into Map, MutableMap, PersistentMap and Set, MutableSet, PersistentSet - [#4989](https://github.com/mozilla/rust/issues/4989)
* Move dlist to std - [#3549](https://github.com/mozilla/rust/issues/3549)
* benchmark whether 1.5x is a better growth factor for vectors - [#4961](https://github.com/mozilla/rust/issues/4961)