# Short answer

`--save-temps` and then use llvm-dis to look at the resulting .bc file.

# Long answer

If the test you want to run is in ''test/run-pass/foobar.rs'', the invocation (as described on the [test suite page][The Rust test suite]) is

`make check TESTNAME=test/run-pass/foobar.rs`

If you stick a ''VERBOSE=1'' in front of that, you'll see what's happening under the hood.  So, for instance:

`VERBOSE=1 make check TESTNAME=test/run-pass/hello.rs`

will spit out a bunch of verbose output, including (if you scroll up a ways) a call to `compiletest`, the test runner, which will look something like:

`stage2/bin/compiletest --compile-lib-path stage2/lib --run-lib-path stage2/lib/rustc/i686-unknown-linux-gnu/lib --rustc-path stage2/bin/rustc --stage-id stage2 --rustcflags "-O" test/run-pass/hello.rs --verbose  --src-base ../rust/src/test/run-pass/ --build-base test/run-pass/ --mode run-pass --runtool "/usr/bin/valgrind --leak-check=full --error-exitcode=100 --quiet --suppressions=../rust/src/etc/x86.supp"`

Beware -- there are a bunch of calls to `compiletest` that get spit out, and most of them are not what you want.

What you want is to add the `--save-temps` flag to the `--rustcflags` list, and the test runner will pass it along to the compiler.  So the entire invocation, for this particular test, becomes:

`stage2/bin/compiletest --compile-lib-path stage2/lib --run-lib-path stage2/lib/rustc/i686-unknown-linux-gnu/lib --rustc-path stage2/bin/rustc --stage-id stage2 --rustcflags "-O --save-temps" test/run-pass/hello.rs --verbose  --src-base ../rust/src/test/run-pass/ --build-base test/run-pass/ --mode run-pass --runtool "/usr/bin/valgrind --leak-check=full --error-exitcode=100 --quiet --suppressions=../rust/src/etc/x86.supp"`

When you do that, you'll see the test runner actually invoke the Rust compiler go by, which will look like

`executing stage2/bin/rustc ../rust/src/test/run-pass/hello.rs -o test/run-pass/hello.stage2 -O --save-temps`

Since the `--save-temps` is in there, an LLVM bitcode file will be sitting in `test/run-pass/hello.bc`.  Run it through `llvm-dis`:

`llvm-dis test/run-pass/hello.bc`

This will produce a `.ll` file that you can look at:

`less test/run-pass/hello.ll`
