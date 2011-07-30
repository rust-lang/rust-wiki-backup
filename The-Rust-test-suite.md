# The Rust test suite

The rust test suite has several sets of tests for different purposes. As the compiler is built several times, over multiple stages, the tests can genarally be run against each stage of the build. Under linux, all tests are run under valgrind if it is installed, unless the build was configured with --disable-valgrind or CFG_DISABLE_VALGRIND=1 is provided on the command line.

### Recipes

* Run the test suite: `make check`. This runs all stage2 tests. This is the criteria for a successful build.
* Run only xfailed (ignored) tests: `make check CHECK_XFAILS=1`
* Run with verbose output: `make check VERBOSE=1`. This will print useful information like the exact commands being run.
* Run without valgrind: `make check CFG_DISABLE_VALGRIND=1`
* Run a specific test: `make check TESTNAME=[...]` - Note that while this will run only tests matching the given pattern, it will still execute all test runners - most of them will just not execute any tests. For more precise control, call make on one of the targets below.
* Run without parallelism: `RUST_THREADS=1 make check` - This can make it easier to understand failure output.

These options can be combined.  For instance, `make check CHECK_XFAILS=1 TESTNAME=test/run-pass/foobar.rs` runs the xfailed test `foobar.rs` in the `run-pass` directory.

## Compiler tests

These are tests of the compiler as a whole. Each test is a source file or source crate over which the compiler is run in some configuration and the resulting executable run. They typically have a `main` function that takes no arguments and may have directives that instruct the test runner how to run the test.

A typical test might look like:

```
// error-pattern:mismatched types
// Regression test for issue #XXX

fn main() {
   let bool a = 10;
}
```

Valid directives include:

* `error-pattern:[...]` - A message that should be expected on standard out. If multiple patterns are provided then they must all be matched, in order.
* `compile-flags:[...]` - Additional arguments to pass to the compiler
* `xfail-stage[N]` - Don't run the test in a specific build stage
* `xfail-fast` - Don't run as part of check-fast, a special win32 test runner (some tests don't work with it)

There are four sets of this type of test:

* run-pass - Tests that are expected to compile and run successfully. They live in src/test/run-pass and their build target is `check-stage[N]-rpass`.
* run-fail - Tests that are expected compile but fail when run. They live in src/test/run-fail and their build target is `check-stage[N]-rfail`. These tests must include at least one 'error-pattern' directive.
* compile-fail - Tests that are expected not to compile. They live in src/test/compile-fail and their build target is `check-stage[N]-cfail`. These tests must include at least one 'error-pattern' directive.
* bench - Benchmarks and miscellaneous snippets of code that are expected to compile and run successfully. They live in src/test/bench and their build target is `check-stage[N]-bench`.

The test runner for these tests is at src/test/compiletest and is compiled to test/compiletest.stage[N].

### Recipes

* Running the run-pass tests for stage1: `make check-stage1-rpass`
* Running a specific compile-fail test: `make check-stage2-cfail TESTNAME=type-mismatch`
* Finding the command to compile a failing test: `make check-stage1-rpass TESTNAME=hello VERBOSE=1`
* Running xfailed tests: `make check-stage1-rpass CHECK_XFAILS=1`

## Compiler unit tests

These are unit tests against rustc internals. They are part of the rustc crate, and use the standard Rust test runner. Their test runner is compiled to test/rustctest.stage[N]

Unit test for rustc go near the code they test and should be wrapped in a conditionally-compiled test module like so

```
#[cfg(test)]
mod test {
   #[test]
   fn resolve_fn_types() {
     ...
   }
}
```

Note that currently tests need to be visible at the top level of a create in order to run.

### Recipes
* Running the tests: `make check-stage[N]-rustc`
* Running a specific test: `make check-stage[N]-rustc TESTNAME=[test]`
* Running ignored tests: `make check-stage[N]-rustc CHECK_XFAILS=1`

## Standard library unit tests

Tests of the standard library live in a separate stdtest crate, under the test/stdtest directory. The compiled test runner lives at test/stdtest.stage[N]

### Recipes
* Running the tests: `make check-stage[N]-std`
* Running a specific test: `make check-stage[N]-std TESTNAME=[test]`
* Running ignored tests: `make check-stage[N]-std CHECK_XFAILS=1`

## Fast check

Because Windows has slow process spawning running `make check` on that platform can take a long time. For this reason we have a `make check-fast` target that the Windows build servers run to keep the cycle time down. This is a special test runner that is built by combining all the run-pass tests into a single library. It is created by the src/etc/combine-tests.py script.