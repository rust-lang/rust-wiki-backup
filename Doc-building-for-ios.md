This currently does not work.

See some approaches: https://github.com/mozilla/rust/issues/6170

## Options

### Option 1: C-code from Rust's LLVM bitcode

```
rustc foo.rs -o foo.stage2 -O --save-temps
llc -march=c foo.bc -o foo.c
```

1. Create `foo.c` from `foo.ll`:
TODO: something using `llc`

### Option 2: Rust's Android-ARM assembler code

TODO: something using `rustc --emit-llvm -S --target=arm-linux-androideabi`

### Option 3: Native iOS-support

1. add an `arm-apple-darwin` target triple to [mk/platform.mk](https://github.com/mozilla/rust/blob/master/mk/platform.mk):
```
# arm-apple-darwin configuration
CFG_IOS_TOOLS=/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/bin/
CFG_IOS_FLAGS = -isysroot /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/ -I /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include -I /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/c++/4.2.1 -I /usr/include
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
1. edit [mk/rt.mk](https://github.com/mozilla/rust/blob/master/mk/rt.mk)?
1. Build rust:
```
mkdir build && cd build
../configure --target-triples=arm-apple-darwin
make
```

This fails with
```
compile: arm-apple-darwin/rt/stage2/sync/lock_and_signal.o
In file included from /Users/lenny/Github/rust_iphone/src/rt/sync/lock_and_signal.cpp:12:
In file included from /Users/lenny/Github/rust_iphone/src/rt/sync/../rust_globals.h:44:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/stdlib.h:63:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/_types.h:27:
In file included from /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/sys/_types.h:32:
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS7.0.sdk/usr/include/sys/cdefs.h:655:2: error: Unsupported architecture
#error Unsupported architecture
```

1. Compile code:
```
rustc --target=arm-apple-darwin foo.rs
```

## Resources

* [Building a GHC cross-compiler for Apple iOS targets](http://ghc.haskell.org/trac/ghc/wiki/Building/CrossCompiling/iOS)
* [[Doc building for android]]
* [iOS ABI Function Call Guide](https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)
* [llc](http://llvm.org/docs/CommandGuide/llc.html)
* [llvm-dis](http://llvm.org/docs/CommandGuide/llvm-dis.html)
* [[Note seeing LLVM output from rust]]