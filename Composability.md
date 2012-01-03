We should think about how to improve composability in Rust.

## Examples
* We should have ```str::any()``` and ```list::any()``` like ```vec::any()```.
But why duplicate the implementation when they all simply expect just a stream of values?

* ```vec::any2()``` is like ```vec::any()``` operating on two lists instead of one. In Haskell you would simply would do ```any foo $ zip ls``` and would not need a special function ```any2```.
In Rust ```vec::zip()``` creates a whole new vector, but we'd need something like ```vec::zip_iter()``` that only iterates over the zipped-list => generates a stream of tuples.


See also http://smallcultfollowing.com/babysteps/blog/2011/12/29/composing-blocks/