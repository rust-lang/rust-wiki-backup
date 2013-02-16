Containers in the standard libraries:

* `core::dlist` (doubly-linked list - uses @ but avoiding that would require `unsafe`)
* `core::vec` (dynamic array)
* `core::str` (string implemented on top of the same dynamic array as `core::vec`)
* `core::hashmap` (hash table - set and map)
* `std::treemap` (balanced binary search tree - set and map)
* `std::fun_treemap` (persistent unbalanced binary search tree - not very useful)
* `std::smallintmap` (dense array-based map)
* `std::deque` (ring buffer)
* `std::priority_queue` (binary heap)
* `std::list` (persistent singly-linked list)

Wanted:

* radix trie (IntMap, IntSet - perhaps more generic)
* b-tree (either in addition to `std::treemap` or replacing it)
* small vector (3-word struct storing small arrays on the stack)
* LRU cache (doubly-linked list and a hash table of key->index)
* persistent balanced binary search tree
* persistent heap
* persistent hash-based map