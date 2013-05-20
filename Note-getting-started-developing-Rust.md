This page describes how to download and build the Rust compiler and
associated tools and libraries from the current git sources.  If
you're more interested in _using_ Rust than in hacking on the Rust
compiler, you might prefer to install a released version instead of
following these instructions.

## Prerequisites

Version numbers listed here are "what we're using"; the code may well work with earlier versions of these tools, but we don't know minimum version requirements.

* A recent Linux, OS X 10.6 or later, Win32 or FreeBSD system
* 2.5 GiB RAM _available for the build process_ (see note below about memory usage)
* [Python 2.x](http://www.python.org/download/) (version 2.7 is known to work)
* GNU make 3.81
* git 1.7
* g++ 4.4 at least on Linux, 4.5 on Windows, and the 4.x gcc in Apple's SDK for OS X.
* curl
* Valgrind 3.5 or later (recommended, but not required for Linux)
* [pandoc](http://johnmacfarlane.net/pandoc/) 1.8 at least (optional, if you wish to build HTML docs)
* pdflatex (optional, if you wish to build PDF docs)
* ccache ([[optional|Note ccache]])

### Memory usage

Rust seems to take about 1.4-1.6 GiB for most of the build, peaking at around 2.5 GiB.  If you only have 2 GiB of RAM total, you may have a difficult time building Rust while doing anything else, such as using a browser.  (One very patient user reported being able to build Rust on a netbook with 1 GiB RAM, but the full three-stage build took six hours.)

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

#### Quick Steps for Windows environment setup.
We recommend developing under [MSYS and MinGW](http://www.mingw.org) using their auto-installer.

1. Download the latest mingw-get-inst (the auto-installer - Green button) directly from http://sourceforge.net/projects/mingw/.
2. During the MinGW installation, check the boxes to select the following components to install: C compiler, C++ compiler, & scroll down and check the box for the MSYS Basic System.
3. After installing MinGW/MSYS, Start the MSYS shell console from the desktop or Start menu.
4. Use the command `mingw-get upgrade "gcc<4.6"` to ensure that GCC is upgraded to the latest version BELOW 4.6.  Do the same for the C++ compiler with the command `mingw-get upgrade "g++<4.6"`.
4. Install Perl with `mingw-get install msys-perl`.
5. Install Curl with ... placeholder for figuring out how to easily install Curl for MinGW Windows users.

#### Further info for Windows users.
You can update MinGW components once you start it's console by using the command `mingw-get update`, this updates the package repository for MinGW.  After which you can upgrade packages with `mingw-get upgrade`.
Using `mingw-get` alone will open a GUI interface for package management.  If you are a consistent user of MinGW or plan to be, you might also want to subscribe to their mailing list: [Mingw-users](https://lists.sourceforge.net/lists/listinfo/mingw-users)

**You currently need to ensure that you are using GCC version < 4.6. for the LLVM / Rust build to complete successfully.**
This is a requirement on Windows 64 bit for LLVM to compile correctly, according to their docs.

Rust will download a git submodule for LLVM during the build and compile it, so you do not need to download LLVM and build it yourself.

Once installed, we tend to work inside the MSYS shell.

(OPTIONAL - if not using MinGW base install, which includes Git) --

For Git, we recommend [MsysGit](http://msysgit.github.com/) and if you use that you will want to put the git binary path *after* the MinGW path. So add a line like the following to your `.bashrc`:

    export PATH=$PATH:/c/Program\ Files/Git/bin

If while building you receive an error that `libpthread-2.dll` is not found, you need to install the [libpthread-2.8.0-3-mingw32-dll-2.tar.lzma package](http://sourceforge.net/projects/mingw/files/MinGW/Base/pthreads-w32/pthreads-w32-2.8.0-3/).  It seems this must be installed by hand, as far as I can tell:

    cd /mingw; lzma -d -c /path/to/downloaded/libpthread-2.8.0-3-mingw32-dll-2.tar.lzma | tar xf -

### OSX

Get the command line tools for xcode from  [Apple Developer Downloads](https://developer.apple.com/downloads/index.action) or Xcode.app -> Preferences -> Downloads -> Components.

Then, optionally get Valgrind and pandoc. Vallgrind is available in machomebrew: `brew install valgrind`. pandoc must be installed manually according to the [installation instructions](http://johnmacfarlane.net/pandoc/installing.html). ccache can also be installed from machomebrew: `brew install ccache`.

### FreeBSD

Rust builds on FreeBSD but is not officially supported. It is only known to work on 9.0-RELEASE. You'll need some prerequisites:

    pkg_add -r git gmake libexecinfo libunwind

The gcc included with FreeBSD is old, so your best bet is to run the `configure` script with `--enable-clang`. Installing gcc 4.6 can also work. Build with `gmake` instead of `make`.

### Android

See [[building for Android|Doc-building-for-android]]

## Downloading and building Rust

    git clone git://github.com/mozilla/rust.git
    cd rust
    ./configure   # this may take a while if this is your first time, as it downloads LLVM
    make    # this will definitely take a while if this is your first time, as it builds LLVM

Optional steps:

    make check   # run the test suite
    make install   # install the compiler and associated tools

This will build and test the compiler, standard libraries, and supporting tools.

*Note:* You can use `make -j8` (if you have an 8-core machine) to speed up the build (at least the LLVM part and the tests). On Linux or OS X, if you have Valgrind installed, the tests will run slowly because they are running under Valgrind. If you define `CFG_DISABLE_VALGRIND=1` in your build environment or run configure with the `--disable-valgrind` flag, you can see the tests running at full speed.

If you are going to be hacking on the Rust compiler itself then it is recommended that you configure with `--disable-optimize`, since this will greatly reduce up your compilation time.

## Navigating

There's a quick guide to the source of the compiler in [src/librustc/README.txt](https://github.com/mozilla/rust/blob/master/src/librustc/README.txt). You should look through it if you're going to be contributing.

## Editor support

Syntax highlighting for vim is included in the Rust repository, under `src/etc/vim` and for emacs under `src/etc/emacs`. Other editors may [[also be supported|Doc packages, editors, and other tools]].

## The issue tracker

We use the [GitHub issue tracker](https://github.com/mozilla/rust/issues) to track bugs and feature requests in Rust.  If you prefer not to use the standard GitHub issue tracker, there's a [secondary front-end that is quite a bit more responsive](http://githubissues.heroku.com/#mozilla/rust).

## Picking something interesting to do

To get an idea of where we're going, see the [[development roadmap|Note development roadmap]].

The way most people seem to get involved is by simply trying to write Rust code. Inevitably you will hit a bug or missing feature, at which point you may feel compelled to write a patch.

Another way to get involved is to look through the issue tracker for issues labeled with "[E-easy]", "[I-enhancement]", "[I-wishlist]", and/or "[A-an-interesting-project]". Unassigned bugs are always fair game. Assigned bugs that don't seem to be getting worked on actively can be fair game, but always check with the bug owner first in that case. Also see the [[development policy|Note development policy]].

[E-easy]: https://github.com/mozilla/rust/issues?labels=E-easy&page=1&state=open
[I-enhancement]: https://github.com/mozilla/rust/issues?labels=I-enhancement&page=1&state=open
[I-wishlist]: https://github.com/mozilla/rust/issues?labels=I-wishlist&page=1&state=open
[A-an-interesting-project]: https://github.com/mozilla/rust/issues?labels=A-an-interesting-project&page=1&state=open

Outstanding bugs or feature requests in Rust often have a corresponding test in the test suite that doesn't yet pass.  One good way to jump into Rust development is to look for files in the test/run-pass directory containing the string `xfail-test`.  Those tests all correspond to bugs that need to be fixed or features that someone needs to finish.

The source is also littered with hundreds of comments marked with 'FIXME' and 'TODO'. In Rust, FIXME comments come with an issue number; sometimes these refer to issues that may be resolved easily.

If in doubt, ask on IRC. Somebody will surely have a task that needs doing.

## Communicating

Join irc.mozilla.org #rust for real-time discussion of Rust development.  We try to remain on that channel during working hours in UTC-7 (US Pacific).

Join the [mailing list](https://mail.mozilla.org/listinfo/rust-dev) for longer conversations.

In both cases, please follow the conduct guidelines on the [[development policy|Note development policy]].