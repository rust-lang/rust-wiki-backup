This page describes how to download and build the Rust compiler and
associated tools and libraries from the current git sources.  If
you're more interested in _using_ Rust than in hacking on the Rust
compiler, you might prefer to install a prebuilt snapshot from [here](http://www.rust-lang.org/install.html), following
the instructions in the [guide](http://doc.rust-lang.org).
For Windows users, see [[Using Rust on Windows]].

## Prerequisites

Version numbers listed here are "what we're using"; the code may well work with earlier versions of these tools, but we don't know minimum version requirements.

* A recent Linux, OS X (10.6 or later), Win32 or FreeBSD system
* 1.5 GiB RAM _available for the build process_ (see note below about memory usage)
* [`python` 2.x](http://www.python.org/download/) (version 2.7 is known to work)
* GNU `make` 3.81
* `git` 1.7
* `g++` 4.7 at least on Linux, 4.5 on Windows, and the 4.x `gcc` in Apple's SDK for OS X.
* `curl`
* recommended: `valgrind` 3.8.0 or later (not required for Linux)
* optional: if you wish to build LaTeX PDF docs, [`pandoc`](http://johnmacfarlane.net/pandoc/) 1.9.1 at least, with one or more of `pdflatex`/`xelatex`/`lualatex`
* optional: `ccache` to shorten your rebuild times

### Memory usage

The Rust build peaks at around 1.3 GiB, so at least that much memory should be available for the build in order to avoid excessive swapping or out-of-memory. If you only have 2 GiB of RAM total, you may have a difficult time building Rust while doing anything else, such as using a browser.

Note that if you are compiling Rust from within a virtual OS (e.g. via `Vagrant`) you may need to change how much RAM your VM has access to.

### Debian-based Linux Distributions

You can install all the prerequisites you need to build Rust by running:

~~~~bash
sudo apt-get install python make git g++ curl valgrind
~~~~

For Ubuntu users willing to build LaTeX doc, you will need some more packages:

~~~~bash
sudo apt-get install pandoc texlive-latex-recommended lmodern
~~~~

The `pandoc` that's packaged with certain Linux
distributions (Ubuntu, for instance) is sometimes quite old, 
so make sure that it's at least version 1.9.1 (you can check 
the version by running `pandoc -V`). If your version's too old, follow the building
instructions [here](http://johnmacfarlane.net/pandoc/installing.html).  
Also, make sure you have the `lmodern` font package (see [#3989](https://github.com/mozilla/rust/issues/3989)).

### Windows

The Rust build system depends on a UNIX shell environment, so you'll need MSYS, which provides such an environment on Windows. We currently recommend [MSYS2](http://sourceforge.net/projects/msys2/), which also provides a convenient package manager. You can install the compiler toolchain using MSYS2's package manager.

(MSYS1 is fine if you already have it, but configuring MSYS1 is not an easy job. Also beware of MSYS1 bugs such as [`make -jN` freezing randomly](http://sourceforge.net/mailarchive/message.php?msg_id=29801372).)

MSYS2 provides three environments: `mingw32_shell.bat`, `mingw64_shell.bat` and `msys2_shell.bat`. For the Rust build you should select correct batch file described below.

#### 32-bit build

Execute `mingw32_shell.bat` to run MSYS2 for a 32-bit system. Run the following commands:

    pacman -S mingw-w64-i686-toolchain
    pacman -S base-devel
    pacman -S git

To check the installation was successful, type `gcc -v`. The target should be `i686-w64-mingw32`.

Now everything is ready, `cd` into the Rust source directory and run:

    ./configure
    make

This will start the Rust build.

#### 64-bit build

Execute `mingw64_shell.bat` to run MSYS2 for a 64-bit system. Run the following commands:

    pacman -S mingw-w64-x86_64-toolchain
    pacman -S base-devel
    pacman -S git

To check the installation was successful, type `gcc -v`. The target should be `x86_64-w64-mingw32`.

### OSX

Get the command line tools for Xcode from  [Apple Developer Downloads](https://developer.apple.com/downloads/index.action), or open Xcode and go to Xcode -> Preferences -> Downloads -> Components.

Then optionally install Valgrind and pandoc. Valgrind is available from [homebrew]: `brew install valgrind`. pandoc must be installed manually according to the [installation instructions](http://johnmacfarlane.net/pandoc/installing.html). ccache can also be installed from [homebrew]: `brew install ccache`.

[homebrew]: http://brew.sh/

Sometimes, on OS X, compiling Rust might fail with a "too many open files" error, especially when running `make check`.
 * One solution is to limit the number of concurrent threads during the run, via the environment variable `RUST_THREADS`, e.g. `% RUST_THREADS=2 make check`.
 * Another solution for this is to raise the open file limit on OS X.  One method to achieve this (that has been tested on 10.7.5) is the following:

1. Raise the number of maximum files allowed on the system: `sudo sysctl -w kern.maxfiles=1048600` and `sudo sysctl -w kern.maxfilesperproc=1048576`.  This can be made persistent by adding the following lines to `/etc/sysctl.conf`:

~~~~
kern.maxfiles=1048600
kern.maxfilesperproc=1048576
~~~~

2. Raise the launchd limits: `sudo launchctl limit maxfiles 1048576`.  Can be made persistent by adding `  limit maxfiles 1048576` to `/etc/launchd.conf`.

3. Verify the changes.  If all goes well, `sudo launchctl limit` should show something like this:

~~~~
  [...]
  maxproc     709            1064
  maxfiles    1048575        1048575
~~~~

4. Run the compiler. Note that if the changes aren't made persistent, you need to run as root, since the per-user launchd won't inherit the settings. If you do change the config files, you need to reboot to apply the appropriate settings.

### FreeBSD

Rust will build on FreeBSD, but it's not officially supported. Building is only known to work on 9.0-RELEASE. You'll need some prerequisites, which you can install with:

~~~~bash
pkg_add -r git gmake libexecinfo libunwind
~~~~

The gcc included with FreeBSD is old, so your best bet is to run the `configure` script with `--enable-clang` (i.e. `./configure --enable-clang`). Installing gcc 4.6 can also work. Build with `gmake` instead of `make`.

### Android

See [[building for Android|Doc-building-for-android]]

## Downloading and Building Rust

Before you get started, you should quickly review the [build system notes](https://github.com/rust-lang/rust/blob/master/Makefile.in#L11) which describes the Make targets, among other things.

~~~~bash
git clone git://github.com/mozilla/rust.git
cd rust
~~~~

Make sure prerequisites, such as Python or Perl, can be found via the `PATH` environment variable, otherwise you will get errors during `./configure`.

The `configure` script will pull git submodules if it's the first time.  This may take a while. 

**`--enable-rpath`:** By default, the newly built `rustc` found in `<target>/stage<N>/bin/` will likely have its shared library dependencies pointing to the system-wide library path e.g. `/usr/lib/`.  This could be problematic if you do not intend to `make install` the shared library dependencies to that destination.  Pass `--enable-rpath` to `configure` to make `rustc` link against the newly built libraries in `<target>/stage<N>/bin/`.  Alternatively, you can muck around with `LD_LIBRARY_PATH`.  See [#20835](/rust-lang/rust/issues/20836) #20836 for more details.

To adjust the install locations, pass `--prefix=/path/to/install/dir` to `configure`.

Run `./configure --help` for various other options.

~~~~bash
./configure --enable-rpath
~~~~

Build the compiler, standard libraries, and supporting tools.

You can use `make -j` to run a parallel build utilizing all available cores.  You can also specify the number of cores to use e.g. `make -j8` for an 8-core machine.

~~~~bash
make
~~~~

Optional steps:

Run the test suite. On windows, `make check` may not pass. In any case, `make check-fast` should work (see #4193).

*Note:* On Linux or OS X, if you have Valgrind installed, the tests will run slowly because they are running under Valgrind. If you define `CFG_DISABLE_VALGRIND=1` in your build environment or run configure with the `--disable-valgrind` flag, you can see the tests running at full speed.

~~~~bash
make check
~~~~

Install the compiler and associated tools.  You may need to use `sudo make install` if you do not normally have permission to modify the destination directory.

~~~~bash
make install
~~~~

*Note:* If you need to pass in extra flags to `make`, you can add `RUSTFLAGS=...` to the argument list for `make`. For example, `make check RUSTFLAGS="-Z debug-info"` builds the compiler and runs tests with debug info enabled. 

*Note:* Some make targets are only exercised by `make check-full`.  If you want to see what commands a make invocation is running, you can add `VERBOSE=1` to the argument list for make. Also, if you use make options like `--print-data-base` to see other targets, note that some rules in the database are only generated dynamically.

## Navigating

There's a quick guide to the source of the compiler in [src/librustc/README.txt](https://github.com/mozilla/rust/blob/master/src/librustc/README.txt). You should look through it if you're planning on contributing.

## Editor support

Syntax highlighting for vim is located at the [rust.vim](https://github.com/rust-lang/rust.vim) repository, and syntax highlighting for emacs is located at the [rust-mode](https://github.com/rust-lang/rust-mode) repository. Other editors may [[also be supported|Doc packages, editors, and other tools]].

## The issue tracker

We use the [GitHub issue tracker](https://github.com/mozilla/rust/issues) to track bugs and feature requests in Rust. If you prefer not to use the standard GitHub issue tracker, there's a [secondary front-end](http://githubissues.heroku.com/#mozilla/rust) that is quite a bit more responsive.

## Communicating

Join irc.mozilla.org #rust for real-time discussion of Rust development. We try to remain on that channel during working hours in UTC-7 (US Pacific).

Join the [mailing list](https://mail.mozilla.org/listinfo/rust-dev) for longer conversations.

In both cases, please follow the conduct guidelines on the [[development policy|Note development policy]].