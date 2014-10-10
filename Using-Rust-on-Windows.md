As of release 0.12, Rust distribution for Windows is completely self-contained and includes all dependencies from MinGW toolchain required for building Windows binaries.

Notes:

1. Rust compiler uses `gcc` as a linker driver, and will search for it in the system `PATH` before searching in the private directory.  This means that any other versions found on `PATH` will override the one bundled with Rust, and may cause linking errors if they aren't compatible.  The solution is to strip down the `PATH` in the console session used to launch `rustc`.
2. The 32-bit binaries produced by `rustc` will depend on the shared GCC runtime library, `libgcc_s_dw2-1.dll`, which you will need to distribute along with your program.  You'll find it in the `/bin` directory.  
The 64-bit binaries are dependency-free.

*These instructions cover running Rust from a binary installation. To build Rust see [[further instructions|Note-getting-started-developing-Rust]].*