We should think about how to improve composability in Rust.

## Examples
* ```str``` and ```list``` should have a function ```any``` like ```vec::any```.
But why duplicate the implementation when they all simply expect just a stream of values?

* ```vec::any2``` is like ```vec::any``` operating no two lists. In Haskell you would simlpy would do ```any foo . zip``` and would not need a special ```any2```.
In Rust ```vec::zip``` creates a whole new vector, while we'd need something like ```vec::zip_iter``` that only iterates over the zipped-list => generates a stream of tuples.


See also http://smallcultfollowing.com/babysteps/blog/2011/12/29/composing-blocks/