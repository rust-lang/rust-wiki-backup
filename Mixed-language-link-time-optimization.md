This can be done with or without the runtime enabled, but the example here will disable it in order to demonstrate a very simple optimization enabled by LTO.

main.c

```c
#include <stdint.h>
#include <stdlib.h>

uint32_t *foo();

int main(void) {
    free(foo());
    return 0;
}
```

foo.rs

```rust
#[no_std];
#[allow(ctypes)];

extern {
    fn malloc(size: uint) -> *mut u8;
    fn free(ptr: *mut u8);
    fn abort() -> !;
}

#[lang = "exchange_malloc"]
unsafe fn exchange_malloc(size: uint) -> *mut u8 {
    let ptr = malloc(size);
    if ptr == 0 as *mut u8 {
        abort()
    }
    ptr
}

#[lang = "exchange_free"]
unsafe fn exchange_free(ptr: *mut u8) {
    free(ptr)
}

#[no_mangle]
extern "C" fn foo() -> ~u32 { ~5 }
```

Build LLVM bytecode from the Rust code:

    $ rustc foo.rs --emit-llvm --lib -O

The `--lib` flag is necessary even if Rust defines `main`, to avoid treating
the Rust part as the entire application.

Compile together the C and Rust code with `clang`:

    $ clang foo.bc main.c -flto -O3 -o main

Verify that LLVM eliminated the call to `foo` via inlining and removed the
allocation as a dead store:

    $ gdb main
    (gdb) disassemble main
    Dump of assembler code for function main:
       0x00000000004005b0 <+0>:	xor    %eax,%eax
       0x00000000004005b2 <+2>:	retq   
    End of assembler dump.

To do this with the Rust runtime on, define a regular `main` function in Rust and pass the necessary link flags to `clang`. The necessary switches can be obtained with `rustc -Z print-link-args foo.rs`. In order for segmented stacks to work, `rustc` will likely need to be taught to take LLVM bytecode files as input to combine with the current crate.