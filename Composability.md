We should think about how to improve composability in Rust.

## Example
```str``` and ```list``` should have a ```any``` like ```vec::any```.

Why duplicate the implementation when they all simply expect just a stream of values?

```vec::any2``` is like ```vec::any``` operating no two lists. In Haskell you would simlpy would do ```any foo . zip``` and would not need a special ```any2```.

In Rust ```vec::zip``` creates a whole new vector, while we'd need something like ```vec::zip_iter``` that only iterates over the zipped-list.


See also http://smallcultfollowing.com/babysteps/blog/2011/12/29/composing-blocks/