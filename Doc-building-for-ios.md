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

1. add an `arm-apple-darwin` target triple to [mk/platform.mk](https://github.com/mozilla/rust/blob/master/mk/platform.mk):
```
# arm-apple-darwin configuration
CFG_IOS_TOOLS=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/
CFG_IOS_FLAGS = -target arm-apple-darwin -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/ -I /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include -I /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/4.2.1 -I /usr/include
CC_arm-apple-darwin= $(CFG_IOS_TOOLS)/clang
CXX_arm-apple-darwin= $(CFG_IOS_TOOLS)/clang++
CPP_arm-apple-darwin= $(CFG_IOS_TOOLS)/clang++
AR_arm-apple-darwin= $(CFG_IOS_TOOLS)/ar
CFG_LIB_NAME_arm-apple-darwin=lib$(1).dylib
CFG_LIB_GLOB_arm-apple-darwin=lib$(1)-*.dylib
CFG_LIB_DSYM_GLOB_arm-apple-darwin=lib$(1)-*.dylib.dSYM
CFG_GCCISH_CFLAGS_arm-apple-darwin := -Wall -Werror -g -fPIC $(CFG_IOS_FLAGS)
CFG_GCCISH_CXXFLAGS_arm-apple-darwin := -fno-rtti $(CFG_IOS_FLAGS)
CFG_GCCISH_LINK_FLAGS_arm-apple-darwin := -dynamiclib -lpthread -framework CoreServices -Wl,-no_compact_unwind 
CFG_GCCISH_DEF_FLAG_arm-apple-darwin := -Wl,-exported_symbols_list,
CFG_GCCISH_PRE_LIB_FLAGS_arm-apple-darwin :=
CFG_GCCISH_POST_LIB_FLAGS_arm-apple-darwin :=
CFG_DEF_SUFFIX_arm-apple-darwin := .darwin.def
CFG_INSTALL_NAME_arm-apple-darwin = -Wl,-install_name,@rpath/$(1)
CFG_LIBUV_LINK_FLAGS_arm-apple-darwin =
CFG_EXE_SUFFIX_arm-apple-darwin :=
CFG_WINDOWSY_arm-apple-darwin :=
CFG_UNIXY_arm-apple-darwin := 1
CFG_PATH_MUNGE_arm-apple-darwin := true
CFG_LDPATH_arm-apple-darwin :=
CFG_RUN_arm-apple-darwin=$(2)
CFG_RUN_TARG_arm-apple-darwin=$(call CFG_RUN_arm-apple-darwin,,$(2))
```
1. Adjust `src/rt/rust_builtin.cpp`:
```
#ifdef __APPLE__
#include <TargetConditionals.h>
#endif
#if defined(__APPLE__) && !defined(TARGET_OS_IPHONE)
#include <crt_externs.h>
```

1. Build Rust:
```
mkdir build; cd build
../configure --target-triples=arm-apple-darwin
make VERBOSE=1
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
* [Building a GHC cross-compiler for Apple iOS targets](http://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling/iOS)
* [[Doc building for android]]
* [iOS ABI Function Call Guide](https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)
* [llc](http://llvm.org/docs/CommandGuide/llc.html)
* [llvm-dis](http://llvm.org/docs/CommandGuide/llvm-dis.html)
* [[Note seeing LLVM output from rust]]