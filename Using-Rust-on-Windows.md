As of release 0.10, Rust still depends on GCC for platform linker and C runtime libraries, so you will need to install one before installing Rust itself. Follow these instructions to set up the environment, then run the [Rust installer].

## MinGW

The current recommended way of obtaining Rust's prerequisites is by using the [mingw-w64 installers](http://sourceforge.net/projects/mingwbuilds/files/mingw-builds-install/mingw-builds-install.exe/download) from the [mingw-builds project](https://sourceforge.net/projects/mingwbuilds/files/host-windows/releases/). The official Rust build bots are using such an installer from late March, 2014.

1. Download and run mingw-builds-install.exe.
2. Choose installation options: architecture=x32, threads=posix, exceptions=dwarf.
3. Now just download and run the [Rust installer].
3. Use Rust compiler from mingw-builds terminal (there will be a shortcut in the Start menu), or add `<mingw-builds>\mingw32\bin` directory to your PATH.
4. Verify Rust installation at mingw-builds terminal by typing `rustc --help`

[Rust installer]: http://static.rust-lang.org/dist/rust-nightly-install.exe
