# The Rust test suite

Note: this is what the test suite will look like in the future, not currently.

The rust test suite has several sets of tests for different purposes. As the compiler is built several times, over multiple stages, the tests can genarally be run against each stage of the build. Under linux, all tests are run under valgrind if it is installed, unless the build was configured with --disable-valgrind or CFG_DISABLE_VALGRIND=1 is provided on the command line.

### Recipes

* Run the test suite: `make check`. This runs all stage2 tests and its success is the typical criteria for a successful build
* Run without valgrind: `make check CFG_DISABLE_VALGRIND=1`

## Compiler tests

These are tests of the compiler as a whole. Each test is a source file or source crate over which the compiler is run in some configuration. They live in the src/test directory.

The compiler tests are further divided into four directories: compile-fail, for source which should fail to compile; run-fail, for source which should compile but fail at runtime; run-pass, for source which should compile and run successfully; and bench, for benchmarks.

The behavior of these tests is controlled by directives in comments at the top of each test. Valid directives include:

* `xfail-stage[N]` - Don't run the test in a specific build stage
* `error-pattern: [...]` - For compile-fail and run-fail tests, the error that the test runner should expect to see on standard out

The test runner for these tests is built into the makefiles.

### Recipes

* Running a specific test: `make test/run-pass/hello.stage1.out`
* Resetting a test result: `rm test/run-pass/hello.stage1.out`
* Adding a test: put a .rs file in the appropriate directory and add the appropriate comment headers

## Compiler unit tests

These are unit tests against rustc internals. They are part of the rustc crate, and use the standard Rust test runner. Their test runner is compiled to test/rustctest.stage[N]

### Recipes
* Running the tests: `make check-stage[N]-rustc`
* Running a specific test: `make check-stage[N]-rustc TESTNAME=[test]`
* Running ignored tests: `make check-stage[N]-rustc CHECK_XFAILS=1`
* Resetting the test results: `rm test/rustctest.stage[N].out`
* Adding a test: tests go directly into rustc, near the functionality they test, and wrapped in a test module:

```
#[cfg(test)]
mod test {
   #[test]
   fn resolve_fn_types() {
     ...
   }
}
```

## Standard library unit tests

Tests of the standard library live in a separate stdtest crate, under the test/stdtest directory. The compiled test runner lives at test/stdtest.stage[N]

## Recipes
* Running the tests: `make check-stage[N]-std`
* Running a specific test: `make check-stage[N]-std TESTNAME=[test]`
* Running ignored tests: `make check-stage[N]-std CHECK_XFAILS=1`
* Resetting the test results: `rm test/stdtest.stage[N].out`
* Adding a test: tests go in test/stdtest, usually in a module named after the std module they test