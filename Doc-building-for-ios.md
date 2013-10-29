This currently does not work.

See some approaches: https://github.com/mozilla/rust/issues/6170

## Options

### Option 1: C-code from Rust's LLVM bitcode

1. Produce a `foo.ll` from a `foo.rs`:
```
rustc foo.rs -o foo.stage2 -O --save-temps
llvm-dis foo.bc
```

1. Create `foo.c` from `foo.ll`:
TODO: something using `llc`

### Option 2: Rust's Android-ARM assembler code

TODO: something using `rustc --emit-llvm -S --target=arm-linux-androideabi`

### Option 3: Native iOS-support

1. add an `arm-apple-darwin` target triple to `mk/platform.mk`
1. TODO

## Resources

* [[Doc building for android]]
* [iOS ABI Function Call Guide](https://developer.apple.com/library/ios/documentation/Xcode/Conceptual/iPhoneOSABIReference/Articles/ARMv6FunctionCallingConventions.html)
* [llc](http://llvm.org/docs/CommandGuide/llc.html)
* [llvm-dis](http://llvm.org/docs/CommandGuide/llvm-dis.html)
* [[Note seeing LLVM output from rust]]