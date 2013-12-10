## How do I convert *X* to *Y*?

### number to string

```rust
let x: int = 42;
let y: ~str = x.to_str();

let a: f64 = 3.14;
let b: ~str = x.to_str();
```
### String to number

```rust
let x: Option<int> = from_str("42");
let y: int = x.unwrap();

let a: Option<f64> = from_str("3.14");
let b: f64 = x.unwrap();
```


# Contributing to this page

For small examples, have full type annotations, as much as is reasonable, to keep it clear what, exactly, everything is doing.