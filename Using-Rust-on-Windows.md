As of release 0.9, Rust still depends on GCC for platform linker and C runtime libraries, so you will need to install one:

## MinGW
The standard way of running Rust on Windows is via the [MinGW/MSYS](http://www.mingw.org/) environment:

1. Click "Downloads" at the top of the page, download and run mingw-get-setup.exe.
2. Once package manager window opens, check the "mingw32-base" option.
3. Optionally, check "msys-base" to install MSYS shell.
4. Apply changes (Installation/Apply Changes).
5. Use Rust compiler from MSYS shell, if you installed it (bonus: rustc will use colors in error messages!).  Or, simply add \<mingw\>\bin to your PATH and use it from the Windows Command Prompt.

Note: For the absolutely minimal installed footprint, don't mark any packages for installation, but simply run `mingw-get install gcc` from the command prompt.


## Alternatives

If you are feeling a bit adventurous, you may also try using a standalone MinGW GCC distribution such as [mingw-builds](http://sourceforge.net/projects/mingwbuilds/):

1. Download and run mingw-builds-install.exe, 
2. Choose installation options: architecture=x32, threads=posix, exceptions=dwarf.
3. Use Rust compiler from mingw-builds terminal (there will be a shortcut in the Start menu), or add \<mingw-builds\>\mingw32\bin directory to your PATH.

