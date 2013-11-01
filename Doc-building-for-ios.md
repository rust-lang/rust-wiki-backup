This currently does not work.

## Options

### Option 1: Convert Rust's LLVM bitcode to C-code

```
rustc foo.rs -o foo.stage2 -O --save-temps
llc -march=c foo.bc -o foo.c
```

### Option 2: Use Rust's Android-ARM assembler code

TODO: something using `rustc --emit-llvm -S --target=arm-linux-androideabi`

### Option 3: Add native iOS-support to Rust

1. Adjust [src/rt/rust_builtin.cpp](https://github.com/mozilla/rust/blob/master/src/rt/rust_builtin.cpp):
```
#ifdef __APPLE__
    #include <TargetConditionals.h>
    #include <mach/mach_time.h>

    #if defined(TARGET_OS_IPHONE)
        extern char **environ;
    #else
        #include <crt_externs.h>
    #endif
#endif
```
and
```
rust_env_pairs() {
#ifdef __APPLE__ && !defined(TARGET_OS_IPHONE)
    char **environ = *_NSGetEnviron();
#endif
```
1. Adjust [src/rt/arch/arm/_context.S](https://github.com/mozilla/rust/blob/master/src/rt/arch/arm/_context.S):
```
.align 2
```
TODO: this needs a guard which makes sures, this is affects only ARM and/or iOS


1. Adjust [src/rt/arch/arm/record_sp.S](https://github.com/mozilla/rust/blob/master/src/rt/arch/arm/record_sp.S):
```
.align 2
```
TODO: this needs a guard which makes sures, this is affects only ARM and/or iOS


1. Build Rust:
```
mkdir build; cd build
../configure --target-triples=arm-apple-darwin
make VERBOSE=1
```

#### TODO
  - remove "-I/usr/include" from mk/platform.mk

Fails with
```
src/libuv/include/uv-errno.h:25:10: fatal error: 'errno.h' file not found
```


1. Use Rust:
```
rustc --target=arm-apple-darwin foo.rs
```

### Possible adjustments

* Compilation:
  * Compile twice, with both `-arch armv7` and `-arch armv7s` (A6 processor) ?
    * See https://github.com/ghc-ios/ghc-ios-scripts/blob/master/arm-apple-darwin10-gcc
* Code:
  * [src/librustc/back/arm.rs](https://github.com/mozilla/rust/blob/master/src/librustc/back/arm.rs)
  * [src/rt/arch/arm/record_sp.S](https://github.com/mozilla/rust/blob/master/src/rt/arch/arm/record_sp.S)

## Resources

* https://github.com/mozilla/rust/issues/6170
* [ARM Machine Directives](http://stuff.mit.edu/afs/athena/project/rhel-doc/3/rhel-as-en-3/arm-directives.html): `.align`
* [Building a GHC cross-compiler for Apple iOS targets](http://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling/iOS)
* [[Doc building for android]]
* [environ.7](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man7/environ.7.html): `_NSGetEnviron()`
* [iOS ABI Function Call Guide](https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)
* [llc](http://llvm.org/docs/CommandGuide/llc.html)
* [llvm-dis](http://llvm.org/docs/CommandGuide/llvm-dis.html)
* [[Note seeing LLVM output from rust]]