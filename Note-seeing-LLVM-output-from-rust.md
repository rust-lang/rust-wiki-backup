## General purpose

In your build directory, invoke the Rust compiler with the `--emit=ir` flag. It will look something like:

```
rustc --emit=ir hello.rs
```

An LLVM IR file that you can read will now be sitting in `hello.ll`.

```
less hello.ll
```

--------

### I want to check the test result of the Rust compiler

Recall that if the test you want to run is in `test/run-pass/hello.rs`, the invocation (as described in [[Note-testsuite]] page) is

```
make check TESTNAME=test/run-pass/hello.rs
```

If you stick a `VERBOSE=1` in front of that, you'll see what's happening under the hood.  So, for instance:

```
VERBOSE=1 make check TESTNAME=test/run-pass/hello.rs
```

will spit out a bunch of verbose output, including (if you scroll up a ways) various calls to `compiletest`, the test runner, one of which will look something like:

```
stage2/bin/compiletest --compile-lib-path stage2/lib --run-lib-path stage2/lib/rustc/i686-unknown-linux-gnu/lib --rustc-path stage2/bin/rustc --stage-id stage2 --rustcflags "-O" test/run-pass/hello.rs --verbose  --src-base ../rust/src/test/run-pass/ --build-base test/run-pass/ --mode run-pass --runtool "/usr/bin/valgrind --leak-check=full --error-exitcode=100 --quiet --suppressions=../rust/src/etc/x86.supp"
```

You can manually add the `-C -save-temps` flag to the `--rustcflags` list, and the test runner will pass it along to the compiler.  So the entire invocation, for this particular test, becomes:

```
stage2/bin/compiletest --compile-lib-path stage2/lib --run-lib-path stage2/lib/rustc/i686-unknown-linux-gnu/lib --rustc-path stage2/bin/rustc --stage-id stage2 --rustcflags "-O -C -save-temps" test/run-pass/hello.rs --verbose  --src-base ../rust/src/test/run-pass/ --build-base test/run-pass/ --mode run-pass --runtool "/usr/bin/valgrind --leak-check=full --error-exitcode=100 --quiet --suppressions=../rust/src/etc/x86.supp"
```

After doing that, your LLVM bitcode file will be sitting in `test/run-pass/hello.bc` and you can disassemble it with `llvm-dis` (which may be found in, e.g., `rust/llvm/x86_64-unknown-linux-gnu/Release+Asserts/bin/`).
