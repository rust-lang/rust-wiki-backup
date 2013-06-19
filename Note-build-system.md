Various build targets. Fill me out.

## Triples

Rust is a cross compiler. At various stages of the build a distinction is made between several different logical architectures, each of which is represented as a `target triple`. These triples have three names, which are best thought of from the perspective of the *output* compilers.

* `build` - The triple for the native platform on which the build is being performed. The compiler is always bootstrapped from the build architecture. There is only one build triple.
* `host` - The triples for which the build process produces executable rustc's. 'host' then refers to the host platforms on which the compiler artifacts run. There may be multiple host triples, but the build triple is always in the set of host triples. Each host triple gets its own subdirectory in the build output.
* `target` - The triples for which the produced compilers can in turn compile. The host triples are a subset of the target triples. All host compilers contain libs for all target platforms. Targets may not have compilers that actually run on that platform, e.g. we don't produce a rustc to run on Android.

Common triples are `x86_64-unknown-linux-gnu`, `i686-unknown-linux-gnu`, `x86_64-apple-darwin`, `i686-apple-darwin`, `i686-pc-mingw32`, `arm-linux-androideabi`.

## Make targets

* `all` - The default rule. Builds a complete stage2 compiler, std, and extra for all hosts and targets
* `docs` - Generate HTML documentation for the std and extra libraries from source code comments
* `rustc` - The stage 2 compiler for the build platform with standard and extra libraries
* `rustc-stage1` - The stage1 compiler for the build platform
* `rustc-stage1-H-x86_64-unknown-linux-gnu` - The stage1 compiler for x86_64 linux
* `x86_64-unknown-linux-gnu/stage1/lib/rustc/arm-linux-androideabi/lib/libstd.so` - Build the stage1 standard library for the android *target*, for the x86_64 *host* compiler (Cross x86_64->android).
* `check-stage1-T-x86_64-unknown-linux-gnu-H-x86_64-unknown-linux-gnu-std` - Test stage1 std for x86_64 linux
* `check-stage1-T-i686-unknown-linux-gnu-H-x86_64-unknown-linux-gnu-std` - Test stage1 std for i686 linux targets, built by the x86_64 host compiler (cross compile x86_64->i686)

See also the notes on the Rust [[test suite|Note testsuite]].