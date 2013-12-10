## How do I convert an int to a string?

Use [`ToStr`](http://static.rust-lang.org/doc/master/std/to_str/trait.ToStr.html).

```rust
let x: int = 42;
let y: ~str = x.to_str();
```

## How do I convert a string to an int?

Use [`FromStr`](http://static.rust-lang.org/doc/master/std/from_str/trait.FromStr.html), and its helper function, [`from_str`](http://static.rust-lang.org/doc/master/std/from_str/fn.from_str.html).

```rust
let x: Option<int> = from_str("42");
let y: int = x.unwrap();
```


# Contributing to this page

For small examples, have full type annotations, as much as is reasonable, to keep it clear what, exactly, everything is doing. Try to link to the API docs, as well.