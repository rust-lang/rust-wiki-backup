## How do I convert an int to a string?

```rust
let x: int = 42;
let y: ~str = x.to_str();
```

## How do I convert a string to an int?

```rust
let x: Option<int> = from_str("42");
let y: int = x.unwrap();
```


# Contributing to this page

For small examples, have full type annotations, as much as is reasonable, to keep it clear what, exactly, everything is doing.