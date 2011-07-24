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

Note that attaching the 'test' attribute to a function does not imply the 'cfg(test)' attribute. Test items must still be explicitly marked for conditional compilation (though this could change in the future).

Tests that should not be run can be annotated with the 'ignore' attribute. The existence of these tests will be noted in the test runner output, but the test will not be run.

A test runner built with the '--test' flag supports a limited set of arguments to control which tests are run: the first free argument passed to a test runner specifies a filter used to narrow down the set of tests being run; the '--ignored' flag tells the test runner to run only tests with the 'ignore' attribute.

## Examples

### Typical test run

```
> mytests

running 30 tests
running driver::tests::mytest1 ... ok
running driver::tests::mytest2 ... ignored
...
running driver::tests::mytest3 ... ok

result: ok. 28 passed; 0 failed; 2 ignored
```