# Rust on iOS

You need XCode 5.

Build Rust cross-compiler:
```
mkdir build_ios; cd build_ios
../configure --target=armv7-apple-ios,armv7s-apple-ios,i386-apple-ios,aarch64-apple-ios,x86_64-apple-ios
make
```

Once compilation complete you can use it.

To target device (depending on dest architecture):
```
rustc --target=armv7-apple-ios foo.rs
```

```
rustc --target=armv7s-apple-ios foo.rs
```

```
rustc --target=aarch64-apple-ios foo.rs
```

To target simulator:
```
rustc --target=i386-apple-ios foo.rs
```

```
rustc --target=x86_64-apple-ios foo.rs
```

## What you get

* all Rust superpowers
* exception handling
* LLDB debugging

## Known limitations

* segmented stack is disabled that means no stack protection available

## Resources

* [Sample](https://github.com/vhbit/ObjCrust) of using Rust from Objective C + Makefile for simultaneous compiling for both device and simulator (all architectures)