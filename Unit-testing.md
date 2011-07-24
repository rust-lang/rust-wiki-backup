# Unit testing in Rust

Rust has built in support for simple unit testing. Functions can be marked as unit tests using the 'test' attribute.

```
#[test]
fn return_none_if_empty() {
   ... test code ...
}
```

To run the tests in a crate, it must be compiled with the '--test' flag: `rustc myprogram.rs --test -o myprogram-tests`. Running the resulting executable will run all the tests in the crate. A test is considered successful if its function returns; if fail is called, directly or indirectly, then the test fails.

When compiling a crate with the '--test' flag '--cfg test' is also implied, so that tests can be conditionally compiled.

```
#[cfg(test)]
mod tests {
  #[test]
  fn return_none_if_empty() {
    ... test code ...
  }
}
```

Note that attaching the 'test' attribute to a function does not imply the 'cfg(test)' attribute. Test items must still be explicitly marked for conditional compilation (this could change in the future).