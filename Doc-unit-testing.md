# Unit testing in Rust

Rust has built in support for simple unit testing. Functions can be marked as unit tests using the 'test' attribute.

```
#[test]
fn return_none_if_empty() {
   ... test code ...
}
```

A test function's signature must have no arguments and no return value. To run the tests in a crate, it must be compiled with the '--test' flag: `rustc myprogram.rs --test -o myprogram-tests`. Running the resulting executable will run all the tests in the crate. A test is considered successful if its function returns; if the task running the test fails, through a call to `fail!`, a failed `check` or `assert`, or some other means, then the test fails.

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

Additionally #[test] items behave as if they also have the #[cfg(test)] attribute, and will not be compiled when the --test flag is not used.

Tests that should not be run can be annotated with the 'ignore' attribute. The existence of these tests will be noted in the test runner output, but the test will not be run. Tests can also be ignored by configuration so, for example, to ignore a test on windows you can write `#[ignore(cfg(target_os = "win32"))]`.

Tests that are intended to fail can be annotated with the 'should_fail' attribute.  The test will be run, and if it causes its task to fail then the test will be counted as successful; otherwise it will be counted as a failure.  For example:

```
#[test]
#[should_fail]
fn test_out_of_bounds_failure() {
  let v: [int] = [];
  v[0];
}
```

A test runner built with the '--test' flag supports a limited set of arguments to control which tests are run: the first free argument passed to a test runner specifies a filter used to narrow down the set of tests being run; the '--ignored' flag tells the test runner to run only tests with the 'ignore' attribute.

## Parallelism

By default, tests are run in parallel, which can make interpreting failure output difficult. In these cases you can set the RUST_THREADS environment variable to 1 to make the tests run sequentially.

## Examples

### Typical test run

```
> mytests

running 30 tests
running driver::tests::mytest1 ... ok
running driver::tests::mytest2 ... ignored
... snip ...
running driver::tests::mytest30 ... ok

result: ok. 28 passed; 0 failed; 2 ignored
```

### Test run with failures

```
> mytests

running 30 tests
running driver::tests::mytest1 ... ok
running driver::tests::mytest2 ... ignored
... snip ...
running driver::tests::mytest30 ... FAILED

result: FAILED. 27 passed; 1 failed; 2 ignored
```

### Running ignored tests

```
> mytests --ignored

running 2 tests
running driver::tests::mytest2 ... failed
running driver::tests::mytest10 ... ok

result: FAILED. 1 passed; 1 failed; 0 ignored
```

### Running a subset of tests

```
> mytests mytest1

running 11 tests
running driver::tests::mytest1 ... ok
running driver::tests::mytest10 ... ignored
... snip ...
running driver::tests::mytest19 ... ok

result: ok. 11 passed; 0 failed; 1 ignored
```