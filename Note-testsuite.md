# The Rust test suite

The rust test suite has several sets of tests for different purposes. As the compiler is built over multiple stages, and with varying host and target combinations, debugging and profiling settings, the tests can generally be run in many different ways.

### Recipes

* Run the test suite: `make check`. This runs all stage2 tests. This is the criteria for a successful build.
* Run only xfailed (ignored, broken) tests: `make check CHECK_XFAILS=1`
* Run with verbose output: `make check VERBOSE=1`. This will print useful information like the exact commands being run.
* Run with valgrind: `make check CFG_ENABLE_VALGRIND=1`
* Run a specific test: `make check TESTNAME=[...]`
  * The pattern `[...]` can be a complete path to a test, such as
    `test/run-pass/foobar.rs`; it can also be any substring of a path.
    For instance, `make check TESTNAME=foo` will run all tests that
    have `foo` in some part of their filename.
  * Note that while this will run only tests matching the given
    pattern, it will still execute all test runners - most of them
    will just not execute any tests. For more precise control, call
    `make` on one of the targets below.
* Run without parallelism: `RUST_THREADS=1 make check` - This can make it easier to understand failure output.

These options can be combined.  For instance, `make check CHECK_XFAILS=1 TESTNAME=test/run-pass/foobar.rs` runs the xfailed test `foobar.rs` in the `run-pass` directory.

## Compiler tests

These are tests of the compiler against Rust source code. They typically have a `main` function that takes no arguments and may have directives that instruct the test runner how to run the test. These tests may be compiled and executed, pretty-printed, jitted, etc. depending on the test configuration.

The test runner for these tests is at src/test/compiletest and is compiled to test/compiletest.stage[N].

A typical test might look like:

```
// xfail-pretty 'bool' doesn't pretty-print (#XXX)
// Regression test for issue #YYY

fn main() {
   let a: bool = 10; //~ ERROR mismatched types
}
```

There are four different modes for compile tests. Each test is run under one or more modes:

* compile-fail - The test should fail to compile. Must include at least one expected error.
* run-fail - The test should compile but fail to run. Must include at least one error-pattern directive.
* run-pass - The test should compile and run successfully
* pretty - The test should round-trip through the pretty-printer and then compile successfully

Valid directives include:

* `compile-flags:[...]` - Additional arguments to pass to the compiler
* `pp-exact` - The test should pretty-print exactly as written
* `pp-exact:[filename]` - The pretty-printed test should match the example in `filename`
* `xfail-test` - Test is broken, don't run it
* `xfail-fast` - Don't run as part of check-fast, a special win32 test runner (some tests don't work with it)
* `xfail-pretty` - Test doesn't pretty-print correctly
* `error-pattern:[...]` - A message that should be expected on standard out. If multiple patterns are provided then they must all be matched, in order (**Note:** error-patterns are somewhat deprecated, see the section on Expected Errors below).
* `no-reformat` - Don't reformat the code when running the test through the pretty-printer
* `aux-build:foo.rs` - Compiles an auxiliary library for use in multi-crate tests.  See the section on multi-crate testing below.
* `exec-env:NAME` or `exec-env:NAME=VALUE` - Sets an environment variable during test execution

There are five directories containing compile tests, living in the src/tests directory:

* run-pass - Tests that are expected to compile and run successfully. Also used for pretty-print testing.
* run-fail - Tests that are expected compile but fail when run. Also used for pretty-print testing.
* compile-fail - Tests that are expected not to compile
* bench - Benchmarks and miscellaneous snippets of code that are expected to compile and run successfully. Also used for pretty-print testing.
* pretty - Pretty-print tests
* auxiliary - libraries used in multi-crate testing. See the section on this topic below.

And finally, build targets:

* check-stage[N]-rpass - The tests in the run-pass directory, in run-pass mode
* check-stage[N]-rfail - The tests in the run-fail-directory, in run-fail mode
* check-stage[N]-cfail - The tests in the compile-fail directory, in compile-fail mode
* check-stage[N]-bench - The tests in the bench directory, in run-pass mode
* check-stage[N]-pretty - All the pretty-print tests
* check-stage[N]-pretty-rpass - The tests in the run-pass directory, in pretty mode
* check-stage[N]-pretty-rfail - The tests in the run-fail directory, in pretty mode
* check-stage[N]-pretty-bench - The tests in the bench directory, in pretty mode
* check-stage[N]-pretty-pretty - The tests in the pretty directory, in pretty mode

### Specifying the expected errors and warnings

When writing a compile-fail test, you must specify at least one
expected error or warning message.  The preferred way to do this is to
place a comment with the form `//~ ERROR msg` or `//~ WARNING msg` on
the line where the error or warning is expected to occur.  You may
have as many of these comments as you like.  The test harness will
verify that the compiler reports precisely the errors/warnings that are
specified, no more and no less.  An example of using the error/warning
messages is:
```
// Regression test for issue #XXX

fn main() {
   let a: bool = 10; //~ ERROR mismatched types
   log (debug, b);
}
```
In fact, this test would fail, because there are two errors: the type
mismatch and the undefined variable `b`.  

Sometimes it is not possible or not convenient to place the `//~`
comment on precisely the line where the error occurs. For those cases,
you may make a comment of the form `//~^` where the caret `^`
indicates that the error is expected to appear on the line above.  You
may have as many caret as you like, so `//~^^^ ERROR foo` indicates
that the error message `foo` is expected to be reported 3 lines above
the comment.  We could therefore correct the above test like so:
```
// Regression test for issue #XXX

fn main() {
   let a: bool = 10; //~ ERROR mismatched types
   log (debug, b);
   //~^ ERROR undefined variable `b`
}
```

The older technique for specify error messages is to use an
`error-pattern` directive.  These directives are placed at the top of
the file and each message found in an `error-pattern` directive must
appear in the output.  Using error comments is preferred, however,
because it is a more thorough test: (a) it verifies that the error is
reported on the expected line number and (b) it verifies that no
additional errors or warnings are reported.

### Multi-crate testing

Sometimes it is useful to write tests that make use of more than one crate.  We have limited support for this scenario.  Basically, you can write and add modules into the `src/test/auxiliary` directory. These files are not built nor tested directly.  Instead, you write a main test in one of the other directories (`run-pass`, `compile-fail`, etc) and add a `aux-build` directive at the head of the main test.  When running the main test, the test framework will build the files it is directed to build from the auxiliary directory.  These builds *must* succeed or the test will fail.  You can then include `use` and `import` commands to make use of the byproducts from these builds as you wish.  

An example consisting of two files:
```
auxiliary/cci_iter_lib.rs:
  #[inline]
  fn iter<T>(v: [T], f: fn(T)) {...}

run-pass/cci_iter_exe.rs:
  // aux-build:cci_iter_lib.rs
  extern mod cci_iter_lib;
  fn main() {
    cci_iter_lib::iter([1, 2, 3]) {|i| ... }
  }
```

### Recipes

* Running the run-pass tests for stage1: `make check-stage1-rpass`
* Running a specific compile-fail test: `make check-stage2-cfail TESTNAME=type-mismatch`
* Finding the command to compile a failing test: `make check-stage1-rpass TESTNAME=hello VERBOSE=1`
* Running xfailed tests: `make check-stage1-rpass CHECK_XFAILS=1`

## Unit Tests

Most crates include <a href="https://github.com/mozilla/rust/wiki/Doc-unit-testing">unit tests</a> which are part of the crate they test. These crates are built with the --test flag and run as part of `make check`.

Tests should go near the code they test. If a module's tests require additional helper functions then they should be enclosed in a conditionally-compiled test module:

```
#[cfg(test)]
mod test {
   fn helper_fn() { ... }

   #[test]
   fn resolve_fn_types() {
     ...
   }
}
```

### Build targets

* `make check-stage[N]-std`
* `make check-stage[N]-extra`
* `make check-stage[N]-syntax`
* `make check-stage[N]-rustc`
* `make check-stage[N]-rustdoc`
* `make check-stage[N]-rusti`

## Documentation tests

The build system is able to extract Rust code snippets from documentation and run them using the compiletest driver. Currently the tutorial and reference manual are tested this way. The targets are `make check-stage[N]-doc-tutorial` and `make check-stage[N]-doc-rust`, respectively. There are also several auxiliary tutorials; to run the tests extracted from them, do:

* `make check-stage[N]-doc-tutorial-borrowed-ptr`
* `make check-stage[N]-doc-tutorial-tasks`
* `make check-stage[N]-doc-tutorial-ffi`
* `make check-stage[N]-doc-tutorial-macros`

To run all doc tests use `make check-stage[N]-doc`.

## Fast check

Because Windows has slow process spawning running `make check` on that platform can take a long time. For this reason we have a `make check-fast` target that the Windows build servers run to keep the cycle time down. This is a special test runner that is built by combining all the run-pass tests into a single library. It is created by the src/etc/combine-tests.py script.