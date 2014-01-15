There are two possible routes to support iOS:
* either let Rust generate LLVM bitcode and use the from Xcode
* or try to add crocc-compilation support to Rust.

## a) LLVM Bitcode

See https://github.com/shilgapira/ObjCrust

## b) Cross-compiler

Smoe issues need to be resolved yet, see the [open issues](https://github.com/mozilla/rust/issues?labels=A-iOS&milestone=13&page=1&state=open).

You need XCode 5.

1. Build Rust:
```
mkdir build_ios; cd build_ios
../configure --target=arm-apple-darwin
make VERBOSE=1
```

1. Use Rust:
```
rustc --target=arm-apple-darwin foo.rs
```

### Adjustments

#### Current adjustments

For the current adjustments:
* see for ```arm-apple-darwin``` in [mk/platform.mk](https://github.com/mozilla/rust/blob/master/mk/platform.mk)
* see for ```apple-darwin``` in [mk/rt.mk](https://github.com/mozilla/rust/blob/master/mk/rt.mk)
* in the source files grep for [```__APPLE__```](https://github.com/mozilla/rust/search?q=__APPLE__&ref=cmdform)

#### Possible future adjustments

* Compilation:
  * Compile twice, with both `-arch armv7` and `-arch armv7s` (A6 processor) ?
    * See https://github.com/ghc-ios/ghc-ios-scripts/blob/master/arm-apple-darwin10-clang
* Code:
  * [src/librustc/back/arm.rs](https://github.com/mozilla/rust/blob/master/src/librustc/back/arm.rs)
  * [src/rt/arch/arm/record_sp.S](https://github.com/mozilla/rust/blob/master/src/rt/arch/arm/record_sp.S)

## Resources

* [ARM Machine Directives](https://sourceware.org/binutils/docs/as/ARM-Directives.html):
* [Building a GHC cross-compiler for Apple iOS targets](http://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling/iOS)
* [CMOSS](https://github.com/mevansam/cmoss/blob/master/build-ios/build-all.sh): scripts for building cross-platform c/c++ open source software libraries for iOS and Android.
* [Cross-compilation using Clang](http://clang.llvm.org/docs/CrossCompilation.html)
* [[Doc building for android]]
* [environ.7](https://developer.apple.com/library/mac/documentation/Darwin/Reference/ManPages/man7/environ.7.html): `_NSGetEnviron()`
* [iOS ABI Function Call Guide](https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)
* [llc](http://llvm.org/docs/CommandGuide/llc.html)
* [llvm-dis](http://llvm.org/docs/CommandGuide/llvm-dis.html)
* [[Note seeing LLVM output from rust]]