## How do I convert *X* to *Y*?

### Int to string

Use [`ToStr`](http://static.rust-lang.org/doc/master/std/to_str/trait.ToStr.html).

```rust
let x: int = 42;
let y: ~str = x.to_str();
```

### String to int

Use [`FromStr`](http://static.rust-lang.org/doc/master/std/from_str/trait.FromStr.html), and its helper function, [`from_str`](http://static.rust-lang.org/doc/master/std/from_str/fn.from_str.html).

```rust
let x: Option<int> = from_str("42");
let y: int = x.unwrap();
```

### Int to string, in non-base-10

Use [`ToStrRadix`](http://static.rust-lang.org/doc/master/std/num/trait.ToStrRadix.html).

```rust
use std::num::ToStrRadix;

let x: int = 42;
let y: ~str = x.to_str_radix(16);
```

### String to int, in non-base-10

Use [`FromStrRadix`](http://static.rust-lang.org/doc/master/std/num/trait.FromStrRadix.html), and its helper function, [`from_str_radix`](http://static.rust-lang.org/doc/master/std/num/fn.from_str_radix.html).

```rust
use std::num::from_str_radix;

let x: Option<int> = from_str_radix("deadbeef", 16);
let y: int = x.unwrap();
```

## How do I get the length of a vector?

The [`Container`](http://static.rust-lang.org/doc/master/std/container/trait.Container.html) trait provides the `len` method.

```rust
let u: ~[u32] = ~[0, 1, 2];
let v: &[u32] = &[0, 1, 2, 3];
let w: [u32, .. 3] = [0, 1, 2, 3, 4];

println!("u: {}, v: {}, w: {}", u.len(), v.len(), w.len()); // 3, 4, 5
```


# Contributing to this page

For small examples, have full type annotations, as much as is reasonable, to keep it clear what, exactly, everything is doing. Try to link to the API docs, as well.

Similar documents for other programming languages:
  * http://pleac.sourceforge.net/