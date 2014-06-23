# 2014-06-23

# Attending

luqman, cmr, brson, aturon, acrichto, bjz, niko, nrc, pcwalton

# clone

https://github.com/rust-lang/rust/blob/master/src/libcore/clone.rs

- Notable: `clone` takes `&self`, not `&mut self`
- Concerns: Utility of clone_from. Implemented by Vec.
- Impls for `&'a T`, `&'a [T]`, `&'a str`, primitives
- Impls for `extern "Rust" fn`
- Recommendation: call the whole thing unstable

- Declare these unstable

impl clone for primitives
../src/libcore/tuple.rs:107:            impl<$($T:Clone),+> Clone for ($($T,)+) {
../src/libcore/cell.rs:195:impl<T:Copy> Clone for Cell<T> {
../src/libcore/cell.rs:301:impl<T: Clone> Clone for RefCell<T> {
../src/libcore/clone.rs:42:impl<'a, T> Clone for &'a T {
../src/libcore/clone.rs:48:impl<'a, T> Clone for &'a [T] {
../src/libcore/clone.rs:54:impl<'a> Clone for &'a str {
../src/liballoc/rc.rs:146:impl<T> Clone for Rc<T> {
../src/liballoc/rc.rs:227:impl<T> Clone for Weak<T> {
../src/liballoc/arc.rs:113:impl<T: Share + Send> Clone for Arc<T> {
../src/liballoc/arc.rs:239:impl<T: Share + Send> Clone for Weak<T> {
../src/liballoc/owned.rs:45:impl<T: Clone> Clone for Box<T> {
../src/libcollections/vec.rs:315:impl<T:Clone> Clone for Vec<T> {
../src/libstd/gc.rs:40:impl<T> Clone for Gc<T> {

nmatsakis: cloning for fns is not good. might want to be careful. doesn't handle bound lifetimes correctly. 

```
<'a> fn(x: &'a Foo) -> &'a uint

fn(T) -> U
```

nmatsakis: clone will just fail - has to match to a template. will for for specific lifetimes, but not for generic lifetimes

```
impl<T:Copy> Clone for T {
    fn clone(&self) -> T {
    }
}
```

acrichto: don't think that is possible. with tuples, if first is copyable, second is not,...
nmatsakis: yeah, painful for tuples.
nmatsakis: not crazy about clone for functions

aturon: do we plan on making a second pass?
brson: yes.

## action items

- clone_from experimental
- extern fns experimental
- everything else unstable
- make listed imples unstable

# bool

https://github.com/rust-lang/rust/blob/master/src/libcore/bool.rs

- Remove to_bit
- Move tests into run-pass as basic language tests
- Leave module intact for docs (for now)
- Module is unstable (or experimental if we are unsure about docs)
- make bool module cfg(docs)

# uint

https://github.com/rust-lang/rust/blob/master/src/libcore/num/uint_macros.rs
https://github.com/rust-lang/rust/blob/master/src/libcore/num/int.rs
https://github.com/rust-lang/rust/blob/master/src/libstd/num/int_macros.rs


## action items

- parse_bytes, FromStr, FromStrRadix experimental - may want to use result
- deprecate to_str_bytes
- deprecate ToStrRadix
