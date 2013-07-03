This page serves as a quick reference for certain corner-cases of the language where it's not obvious how to make the compiler stop complaining at you.

## Once functions

**Why can't closures move out of captured variables?**

Consider the following code:
```
fn foo(message: ~str, chan_opt: Option<Chan<~str>>) {
    do chan_opt.map |chan| {
        // error: cannot move out of captured outer variable
        chan.send(message);
        //        ^~~~~~~
    }
}
```
This code should be legal, because the closure provided to ```map_consume``` will only be called once, so it's just as if ```message``` were moved out without a closure being involved. However, the compiler can't distinguish that case from this one:
```
fn foo(message: ~str, chan_list: ~[Chan<~str>]) {
    do chan_list.map |chan| {
        // XXX: might unsafely duplicate the unique message pointer!
        chan.send(message);
    }
}
```
**Recommended solution:** To work around this problem, you could either use an alternate higher-order function that accepts an additional argument to pass through:
```
fn foo(message: ~str, chan_opt: Option<Chan<~str>>) {
    do chan_opt.map_with(message) |message, chan| {
        chan.send(message);
    }
}
```
...or, if no such alternate exists, use the ```std::cell::Cell``` type:
```
fn foo(message: ~str, chan_opt: Option<Chan<~str>>) {
    let message_cell == Cell::new(message);
    do chan_opt.map |chan| {
        // NOTE: If the closure is called twice, the 2nd take() will fail!
        chan.send(message_cell.take());
    }
}
```
We are prototyping a language feature called *one-shot functions*, where you can write the type of a closure as ```&once fn()``` or ```~once fn()```, to indicate that the closure must be consumed when called, so it can safely move out of its environment. Currently you can enable this by compiling with ```rustc -Z once-fns```, but this is not likely to be enabled by default in the 1.0 release.


## Noncopyable stack closures

**Why can't I use my stack closure multiple times (rustc says, "moved by default, use copy to override")?**

You may have seen this error if you have code along the lines of:
```
fn bar(callback: &fn(int)) {
    baz(13, callback);
    callback(37); // error: use of moved value
}
```
or:
```
fn bar(vec: &[int], callback: &fn(int)) {
    for vec.each |x| {
        baz(x, callback); // error: cannot move out of captured outer variable
    }
}
```
The compiler issues this error because while it's safe to *call* ```&fn()```s multiple times, it is not safe to *copy* them. (Allowing that would [break soundness](http://smallcultfollowing.com/babysteps/blog/2013/04/30/the-case-of-the-recurring-closure/).)

**Recommended solution:** To work around this error, "borrow" the closure by capturing it in another closure, like so:
```
fn bar(callback: &fn(int)) {
    baz(13, |x| callback(x)); // callback temporarily captured, but not moved
    callback(37); // ok
}
```
In the future we hope to automatically perform such borrows, so the original code will be legal again.