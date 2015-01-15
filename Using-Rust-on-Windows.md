As of release 0.12, Rust distribution for Windows is completely self-contained and includes all dependencies from MinGW toolchain required for building Windows binaries.

Notes:

1. Rust compiler uses `gcc` as a linker driver, and will search for it in the system `PATH` before searching in the private directory.  This means that any other versions found on `PATH` will override the one bundled with Rust, and may cause linking errors if they aren't compatible.  The solution is to strip down the `PATH` in the console session used to launch `rustc`.
2. The 32-bit binaries produced by `rustc` will depend on the shared GCC runtime library, `libgcc_s_dw2-1.dll`, which you will need to distribute along with your program.  You'll find it in the `/bin` directory.  
The 64-bit binaries are dependency-free.

## Using Rust with MSYS2
If you want to use any dependencies that depend on C/C++ bits you will likely need a separately installed mingw as the mingw bundled with Rust is not capable of compiling C/C++ code, and only includes a minimal set of libs for system libraries. The recommended way to do this is using MSYS2.

1. Download and install MSYS2 from http://msys2.github.io/.
2. Download and install Rust+Cargo using the installer but be sure to disable the `Linker and platform libraries` option.
3. Ensure that Rust is in your `PATH` and that there are no mingw installations in your `PATH`.
4. Launch your MSYS2 environment using either `mingw32_shell.bat` or `mingw64_shell.bat` depending upon which version of Rust you installed.
5. Install the mingw toolchain using `pacman -S mingw-w64-x86_64-toolchain` or `pacman -S mingw-w64-i686-toolchain` depending upon which version of Rust you installed.
6. Install the base set of developer tools using `pacman -S base-devel`.


*These instructions cover running Rust from a binary installation. To build Rust see [[further instructions|Note-getting-started-developing-Rust]].*