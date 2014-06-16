# Rust on iOS

You need XCode 5.

Build Rust cross-compiler:
```
mkdir build_ios; cd build_ios
../configure --target=arm-apple-ios --target=i386-apple-ios
make VERBOSE=1
```

Once compilation complete you can use it.

To target device:
```
rustc --target=arm-apple-ios foo.rs
```

To target simulator:
```
rustc --target=i386-apple-ios foo.rs
```

## What you get

* all Rust superpowers
* exception handling
* LLDB debugging
* green tasks

## Known limitations

* works only for armv7 architecture, armv7s and arm64 will be added soon.
* segmented stack is disabled that means no stack protection available

## Resources

* [Sample](https://github.com/vhbit/ObjCrust) of using Rust from Objective C + Makefile for simulateous compiling for both device and simulator 