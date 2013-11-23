This page describes how to download and build the Rust compiler and
associated tools and libraries from the current git sources.  If
you're more interested in _using_ Rust than in hacking on the Rust
compiler, you might prefer to install a released version, following
the instructions in the [tutorial](http://doc.rust-lang.org/doc/master/tutorial.html).

## Prerequisites

Version numbers listed here are "what we're using"; the code may well work with earlier versions of these tools, but we don't know minimum version requirements.

* A recent Linux, OS X 10.6 or later, Win32 or FreeBSD system
* 1.5 GiB RAM _available for the build process_ (see note below about memory usage)
* [Python 2.x](http://www.python.org/download/) (version 2.7 is known to work)
* GNU make 3.81
* git 1.7
* g++ 4.4 at least on Linux, 4.5 on Windows, and the 4.x gcc in Apple's SDK for OS X.
* curl
* Valgrind 3.8.0 or later (recommended, but not required for Linux)
* [pandoc](http://johnmacfarlane.net/pandoc/) 1.8 at least (optional, if you wish to build HTML docs)
* pdflatex (optional, if you wish to build PDF docs)
* ccache ([[optional|Note ccache]])

### Memory usage

The Rust build peaks at around 1.5 GiB, so at least that much memory should be available for the build to avoid excessive swapping or out-of-memory. If you only have 2 GiB of RAM total, you may have a difficult time building Rust while doing anything else, such as using a browser.

### Debian-based Linux distributions

You can install all the prerequisites you need to build Rust by running

    sudo apt-get install python make git g++ curl valgrind pandoc texlive-latex-recommended

Incidentally, the pandoc that's packaged for certain Linux
distributions (Ubuntu, for instance) is older than 1.8, so in order to
build HTML docs on Ubuntu, you'll need to install pandoc manually
according to the [installation
instructions](http://johnmacfarlane.net/pandoc/installing.html).

For Ubuntu 11.10 there seems to be a conflict with texlive-latex-base, per [#1692](https://github.com/mozilla/rust/issues/1692).

### Windows

**Note:** To build Rust < 0.8 on Windows systems using MSYS, MinGW, and **GCC 4.8** please see this temporary in-progress guide: [[Building Rust Before 0.8 on Windows Systems|Note-Building-Rust-Before-0.8-on-Windows-Systems]]

#### Quick Steps for Windows environment setup.
We recommend developing under [MSYS and MinGW](http://www.mingw.org) using their auto-installer.

1. Download and install Git for Windows following these steps:
 * Download latest [Git for Windows on Google Code](https://code.google.com/p/msysgit/downloads/list?q=full+installer+official+git) and run it.
 * 5 clicks on Next
 * Choose to **Run Git from the Windows Command Prompt** instead of Use Git Bash only.
 * Choose **Checkout as-is, commit Unix-style line endings** (you'll have less problems).
 * After installing Git, close the MinGW console and reopen it, type `git --version` to verify installation and path is set correctly.
2. Download Python 2.7 installer for your Windows version from http://www.python.org/getit/ and install it preferably to `C:\Python27`.
3. Download the latest mingw-get-inst (the auto-installer - Green button) directly from http://sourceforge.net/projects/mingw/.
4. Run the mingw-get-inst-########.exe
5. In the Setup GUI, Check the boxes and _Mark for Installation_ the following components to install:
    * Basic Setup ->
        * mingw32-base  (has GCC C Compiler)
        * mingw32-gcc-g++ (has GCC C++ Compiler)
        * msys-base  (has essential tools, etc)
    * All Packages -> MinGW Libraries -> MinGW Standard Libraries ->
        * mingw32-libpthread-old
        * mingw32-libpthreadgc
    * Menu - Installation ->
        * Apply Changes
6. Create a shortcut on your desktop to Msys for `C:\MinGW\msys\1.0\msys.bat`
7. Launch Msys and type `sh /postinstall/pi.sh`  (use `c:/mingw` when asked).
8. Install Perl with `mingw-get install msys-perl`.
9. Install wget with `mingw-get install msys-wget`.
10. (Do the following sub-steps also , until we bundle old dll's or get a workaround)
 * Get old versions of these dlls:
    * `mingw-get upgrade "g++<4.6"`
    * `mingw-get upgrade "libpthread=2.8.0-3"`
 *  Copy libgcc_s_dw2-1.dll, libstdc++-6.dll and libpthread-2.dll from `%mingw%\bin` into `%build%\i686-pc-mingw32\stage0\bin`
 * Roll mingw back to the latest: `mingw-get upgrade` 
11. You can now start the MinGW / Msys Shell from your Desktop or Start Menu.
12. Scroll down to [Downloading and building Rust](https://github.com/mozilla/rust/wiki/Note-getting-started-developing-Rust#downloading-and-building-rust) section.

#### More info for Windows users.

**You currently need to ensure that you are using GCC version < 4.6. for the LLVM / Rust build to complete successfully.**
This is a requirement on Windows 64 bit for LLVM to compile correctly, according to their docs.

Rust will download a git submodule for LLVM during the build and compile it, so you do not need to download LLVM and build it yourself.

Once installed, we tend to work inside the MSYS shell.
If you are a consistent user of MinGW or plan to be, you might also want to subscribe to their mailing list: [Mingw-users](https://lists.sourceforge.net/lists/listinfo/mingw-users)

(OPTIONAL)
* Using `mingw-get` alone will open a GUI interface for package management.  You can update MinGW components once you start it's console by using the command `mingw-get update`, this updates the package repository for MinGW.  After which you can upgrade packages with `mingw-get upgrade`. **mingw-get upgrade will overwrite to latest versions of GCC as well, so you might have to upgrade to a lower version afterwards.**

(OPTIONAL - if using another Git installer or method other than the Quick Steps) :

* Put the git binary path *after* the MinGW path. So add a line like the following to your `/etc/profile` or `.bashrc`:

    ``export PATH=$PATH:/c/Program\ Files/Git/bin``

(OPTIONAL - working with multiple toolchains & modifying your PATH) :

* In Windows, to try out or work with different toolchains for building Rust with MinGW, the easiest way is to modify and add at the end of your Msys profile file, `C:\MinGW\msys\1.0\etc\profile` something like the following, and uncomment whichever toolchain path you want to work with and Remember to exit the Msys console and reopen it:

``# Enable MinGW64 path by uncommenting one of the following lines:``
``# export PATH="/c/mingw-builds/x64-4.8.1-release-win32-seh-rev5/mingw64/bin:/usr/local:$PATH"``
``# export PATH="/c/drangon_mingw64/mingw64/bin:/usr/local:$PATH"``
``export PATH="/c/mingw-builds/x32-4.8.1-release-win32-dwarf-rev5/mingw32/bin:/usr/local:$PATH"``

``# Enable LLVM path by uncommenting the following line:``
``# export PATH="$HOME/build/Release+Asserts/bin:$PATH"``

**Troubleshooting Windows environment setups:**

* If Python cannot be found during ./configure, then you need to add its path to the end of `/c/mingw/msys/1.0/etc/profile`, like this:
 * `export PATH="/c/PYTHON27:$PATH"`

* If while building you receive an error that `libpthread-2.dll` is not found, you forgot to install the mingw32-libpthread-old package into MinGW from Step 3, so do this:

 * `mingw-get install mingw32-libpthread-old`

* If while building you receive an error that `libpthreadGC.dll` is not found, you forgot to install libpthreadgc runtime library into MinGW from Step 3, so do this:

 * `mingw-get install mingw32-libpthreadgc`

* If while building you receive an error that `libpthreadGC2.dll` is not found (e.g. when llvm-config tries to run), here is one workaround. (There may be a mingw-get solution too... anyone? I've done windows dev but I'm new to mingw):

 * download pthreads for win32, e.g. here ftp://sourceware.org/pub/pthreads-win32/pthreads-w32-2-9-1-release.zip
 * unzip the zip file, and copy the pthreads-w32-2-9-1-release/Pre-built.d/dll/*.dll to c:/mingw/bin/


### OSX

Get the command line tools for xcode from  [Apple Developer Downloads](https://developer.apple.com/downloads/index.action) or Xcode.app -> Preferences -> Downloads -> Components.

Then, optionally get Valgrind and pandoc. Vallgrind is available in machomebrew: `brew install valgrind`. pandoc must be installed manually according to the [installation instructions](http://johnmacfarlane.net/pandoc/installing.html). ccache can also be installed from machomebrew: `brew install ccache`.

Sometimes, on OS X, compiling Rust might fail with a "too many open files" error, especially when running `make check`.
 * One solution is to limit the number of concurrent threads during the run, via the environment variable `RUST_THREADS`, e.g. `% RUST_THREADS=2 make check`.
 * Another solution for this is to raise the open file limit on OS X.  One method to achieve the latter that has been tested on 10.7.5 is the following:

1. Raise the number of maximum files allowed on the system: `sudo sysctl -w kern.maxfiles=1048600` and `sudo sysctl -w kern.maxfilesperproc=1048576`.  This can be made persistent by adding the following lines to `/etc/sysctl.conf`:

        kern.maxfiles=1048600
        kern.maxfilesperproc=1048576

2. Raise the launchd limits: `sudo launchctl limit maxfiles 1048576`.  Can be made persistent by adding `  limit maxfiles 1048576` to `/etc/launchd.conf`.

3. Verify the changes.  If all goes well, `sudo launchctl limit` should show something like this:

        [...]
        maxproc     709            1064
        maxfiles    1048575        1048575

4. Run the compiler.  Note that if the changes aren't made persistent, you need to run as root, since the per-user launchd won't inherit the settings.  If you do change the config files, you need to reboot to apply the appropriate settings.

### FreeBSD

Rust builds on FreeBSD but is not officially supported. It is only known to work on 9.0-RELEASE. You'll need some prerequisites:

    pkg_add -r git gmake libexecinfo libunwind

The gcc included with FreeBSD is old, so your best bet is to run the `configure` script with `--enable-clang`. Installing gcc 4.6 can also work. Build with `gmake` instead of `make`.

### Android

See [[building for Android|Doc-building-for-android]]

## Downloading and building Rust

Before you get started, you should quickly review the [Build system notes here](note-build-system) which describes the Make targets among other things.

    git clone git://github.com/mozilla/rust.git
    cd rust
    ./configure   # this may take a while if this is your first time, as it downloads LLVM

If you already have one of the prereqs installed, like Python or Perl, make sure the PATH environment variable is set so the configure script can find it, otherwise you will get errors during configure.

    make    # this will definitely take a while if this is your first time, as it builds LLVM

Optional steps:

    make check   # run the test suite
    make install   # install the compiler and associated tools

This will build and test the compiler, standard libraries, and supporting tools.

*Note:* You can use `make -j8` (if you have an 8-core machine) to speed up the build (at least the LLVM part and the tests). On Linux or OS X, if you have Valgrind installed, the tests will run slowly because they are running under Valgrind. If you define `CFG_DISABLE_VALGRIND=1` in your build environment or run configure with the `--disable-valgrind` flag, you can see the tests running at full speed.

If you are going to be hacking on the Rust compiler itself, then you may want to configure with `--disable-optimize`. This sometimes reduces compilation time (but sometimes doesn't, so you are advised to do the comparison for yourself).

*Note:* If you need to pass in extra flags to `make`, you can add `RUSTFLAGS=...` to the argument list for `make`. For example, `make check RUSTFLAGS="-Z debug-info"` builds the compiler and runs tests with debug info enabled. 

*Note:* Some make targets are only exercised by `make check-full`.  If you want to see what commands a make invocation is running, you can add `VERBOSE=1` to the argument list for make.  (Also, if you use make options like `--print-data-base` to see other targets, note that some rules in the database are only generated dynamically.)  See also: [Build system notes](note-build-system)

## Navigating

There's a quick guide to the source of the compiler in [src/librustc/README.txt](https://github.com/mozilla/rust/blob/master/src/librustc/README.txt). You should look through it if you're going to be contributing.

## Editor support

Syntax highlighting for vim is included in the Rust repository, under `src/etc/vim` and for emacs under `src/etc/emacs`. Other editors may [[also be supported|Doc packages, editors, and other tools]].

## The issue tracker

We use the [GitHub issue tracker](https://github.com/mozilla/rust/issues) to track bugs and feature requests in Rust.  If you prefer not to use the standard GitHub issue tracker, there's a [secondary front-end that is quite a bit more responsive](http://githubissues.heroku.com/#mozilla/rust).

## Communicating

Join irc.mozilla.org #rust for real-time discussion of Rust development.  We try to remain on that channel during working hours in UTC-7 (US Pacific).

Join the [mailing list](https://mail.mozilla.org/listinfo/rust-dev) for longer conversations.

In both cases, please follow the conduct guidelines on the [[development policy|Note development policy]].