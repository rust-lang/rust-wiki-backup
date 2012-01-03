We should think about how to improve composability in Rust.

## Example
```str``` and ```list``` should have a ```any``` like ```vec::any```.

But why duplicate the implementation when they all simply expect just a stream of values?

See http://smallcultfollowing.com/babysteps/blog/2011/12/29/composing-blocks/