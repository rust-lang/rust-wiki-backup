As of release 0.9, Rust still depends on GCC for platform linker and C runtime libraries.

The standard way of using Rust on Windows is to install [MinGW/MSYS](http://www.mingw.org/) environment:

1. Click "Downloads" at the top of the page, download and run mingw-get-setup.exe.
2. Once MinGW is installed, make sure to install the GCC package by running `mingw-get install gcc` in MSYS shell.
3. Use Rust compiler from MSYS shell, or make sure that $MINGW\bin is on your path, if you prefer Windows command prompt.


If you would like to reduce the install footprint, and are feeling a bit adventurous, you can try a standalone MinGW GCC distribution such as [mingw-builds](http://sourceforge.net/projects/mingwbuilds/):

1. Download and run mingw-builds-install.exe