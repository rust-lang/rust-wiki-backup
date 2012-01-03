We should think about how to improve composability in Rust.

## Examples
* We should have ```str::any()``` and ```list::any()``` like ```vec::any()```.
But why duplicate the implementation when they all simply expect a stream of values?

* ```vec::any2()``` is like ```vec::any()``` operating on two lists instead of one. In Haskell you would simply would do ```any foo $ zip ls``` and would not need a special function ```any2```.
In Rust ```vec::zip()``` creates a whole new vector, but we'd need something like ```vec::zip_iter()``` that only iterates over the zipped-list => generates a stream of tuples.

## Proposal
Why were iterators [ruled out](http://smallcultfollowing.com/babysteps/blog/2011/12/29/composing-blocks/)?
I think they would be ideal for composability.

* If we had something like ```str::iterator() -> iterator<T>```, ```list::iterator() -> iterator<T>```, ```vec::iterator() -> iterator<T>```
* we would only need one ```any(iterator<T>) -> bool```
* and could get rid of duplicate ```str::any()```, ```list::any()``` and ```vec::any()``` as well any ```vec::any2()```