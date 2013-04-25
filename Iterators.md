The standard library has two iteration modules: `core::iter` for internal iteration, and `core::iterator` for external iteration. The modules will define adaptor algorithms (`filter`, `zip`, etc.) for implementations of either iteration protocol. The adaptors will implement the same iteration protocol, making them composable. The modules will also define functions to consume iterators like `fold` and `to_vec`.

Internal iterators are functions implementing the protocol used by the `for` loop. An internal iterator takes `fn(...) -> bool` as the last parameter, with returning `false` used to
signal breaking out of iteration. The adaptors in the module should work with any such iterator, not just
ones tied to specific traits. For example:

```rust
use core::iter::iter_to_vec;
println(iter_to_vec(|f| uint::range(0, 20, f)).to_str());
```

The old `iter` module defined the adaptors as working on `BaseIter`, which crippled them by restricting the usage to an `each` method on a container taking `&self`. Right now `iter_to_vec` is the only function implementing the composable way, but it's easy enough to redefine the others.

An external iterator object implementing the interface in the `iterator` module can be used as an
internal iterator by calling the `advance` method. For example:

```rust
use core::iterator::*;

let xs = [0u, 1, 2, 3, 4, 5];
let ys = [30, 40, 50, 60];
let mut it = xs.iter().chain(ys.iter());
for it.advance |&x: &uint| {
    println(x.to_str());
}
```

Internal iterators provide a subset of the functionality of an external iterator since the caller cannot store the state and iterate as-needed. It's not possible
to interleave them to implement algorithms like `zip`, `union` and `merge`. However, they're often
much easier to implement.

External iterator adaptors are defined as methods on any `Iterator` implementation returning a state machine implementing `Iterator`. In the future they can be switched to default methods instead of a utility trait. The module currently defines these adaptors:

```rust
    fn chain(self, other: Self) -> ChainIterator<Self>;
    fn zip<B, U: Iterator<B>>(self, other: U) -> ZipIterator<Self, U>;
    // FIXME: #5898: should be called map
    fn transform<'r, B>(self, f: &'r fn(A) -> B) -> MapIterator<'r, A, B, Self>;
    fn filter<'r>(self, predicate: &'r fn(&A) -> bool) -> FilterIterator<'r, A, Self>;
    fn enumerate(self) -> EnumerateIterator<Self>;
    fn skip_while<'r>(self, predicate: &'r fn(&A) -> bool) -> SkipWhileIterator<'r, A, Self>;
    fn take_while<'r>(self, predicate: &'r fn(&A) -> bool) -> TakeWhileIterator<'r, A, Self>;
    fn skip(self, n: uint) -> SkipIterator<Self>;
    fn take(self, n: uint) -> TakeIterator<Self>;
    fn scan<'r, St, B>(self, initial_state: St, f: &'r fn(&mut St, A) -> Option<B>)
        -> ScanIterator<'r, A, B, Self, St>;
    fn advance(&mut self, f: &fn(A) -> bool);
```

Other than naming, that should stay the same indefinitely.