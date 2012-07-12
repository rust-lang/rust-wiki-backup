This page describes how to download and build the Rust compiler and
associated tools and libraries from the current git sources.  If
you're more interested in _using_ Rust than in hacking on the Rust
compiler, you might prefer to install a released version instead of
following these instructions.

## Prerequisites

Version numbers listed here are "what we're using"; the code may well work with earlier versions of these tools, but we don't know minimum version requirements.

* A recent Linux, OS X, Win32 or FreeBSD system
* 1 GB RAM
* [Python 2.x](http://www.python.org/download/) (version 2.7 is known to work)
* GNU make 3.81
* git 1.7
* g++ 4.4 at least on Linux, 4.5 on Windows, and the 4.x gcc in Apple's SDK for OS X.
* curl
* Valgrind 3.5 or later (recommended, but not required for Linux)
* [pandoc](http://johnmacfarlane.net/pandoc/) 1.8 at least (optional, if you wish to build HTML docs)
* pdflatex (optional, if you wish to build PDF docs)

### Debian-based Linux distributions

You can install all the prerequisites you need to build Rust by running

    sudo apt-get install python make git g++ curl valgrind pandoc texlive-latex-recommended

Incidentally, the pandoc that's packaged for certain Linux
distributions (Ubuntu, for instance) is older than 1.8, so in order to
build HTML docs on Ubuntu, you'll need to install pandoc manually
according to the [installation
instructions](http://johnmacfarlane.net/pandoc/installing.html).

### Windows

We recommend developing under [MSYS and MinGW](http://www.mingw.org) using their auto-installer. Be careful with versions: the version dated [20110802](http://sourceforge.net/projects/mingw/files/Installer/mingw-get-inst/mingw-get-inst-20110802/mingw-get-inst-20110802.exe/download) is known to work; later versions including gcc 4.7 are known _not_ to work. Once installed, we tend to work inside the MSYS shell.

For Git, we recommend [MsysGit](http://code.google.com/p/msysgit/) and if you use that you will want to put the git binary path *after* the MinGW path. So add a line like the following to your `.bashrc`:

    export PATH=$PATH:/c/Program\ Files/Git/bin

If while building you receive an error that `libpthread-2.dll` is not found, you need to install the [libpthread-2.8.0-3-mingw32-dll-2.tar.lzma package](http://sourceforge.net/projects/mingw/files/MinGW/Base/pthreads-w32/pthreads-w32-2.8.0-3/).  It seems this must be installed by hand, as far as I can tell:

    cd /mingw; lzma -d -c /path/to/downloaded/libpthread-2.8.0-3-mingw32-dll-2.tar.lzma | tar xf -

### FreeBSD

Rust builds on FreeBSD but is not officially supported. It is only known to work on 9.0-RELEASE. You'll need some prerequisites:

    pkg_add -r git gmake libexecinfo libunwind

The gcc included with FreeBSD is old, so your best bet is to run the `configure` script with `--enable-clang`. Installing gcc 4.6 can also work. Build with `gmake` instead of `make`.

## Downloading and building Rust

    git clone git://github.com/mozilla/rust.git
    cd rust
    mkdir build
    cd build
    ../configure
    make check

This will build and test the compiler, standard libraries, and supporting tools.

*Note:* You can use `make -j8` (if you have an 8-core machine) to speed up the build (at least the LLVM part and the tests). On Linux or OS X, if you have Valgrind installed, the tests will run slowly because they are running under Valgrind. If you define `CFG_DISABLE_VALGRIND=1` in your build environment or run configure with the `--disable-valgrind` flag, you can see the tests running at full speed.

If you are going to be hacking on the Rust compiler itself then it is recommended that you configure with `--disable-optimize`, since this will greatly speed up your compilation. If you use `--enable-debug`, expect some test failures in class-related tests ([bug #2194](https://github.com/mozilla/rust/issues/2194)).

## Packages

Rust is also packaged for various systems by community members:

* [Ubuntu PPA](https://launchpad.net/~kevincantu/+archive/rust/) - maintained by kcantu
* [FreeBSD Port](http://www.freebsd.org/cgi/cvsweb.cgi/ports/lang/rust/) - maintained by jyyou

## Navigating

There's a quick guide to the source of the compiler in [src/rustc/README.txt](https://github.com/mozilla/rust/blob/master/src/rustc/README.txt). You should look through it if you're going to be contributing.

## Editor support

Syntax highlighting for vim is included in the Rust repository, under `src/etc/vim` and for emacs under `src/etc/emacs`. Support for other editors can be found on [[Note related projects]].

## The issue tracker

We use the [GitHub issue tracker](https://github.com/mozilla/rust/issues) to track bugs and feature requests in Rust.  If you prefer not to use the standard GitHub issue tracker, there's a [secondary front-end that is quite a bit more responsive](http://githubissues.heroku.com/#mozilla/rust) and a [tertiary front-end that is pleasantly minimal](http://izs.no.de/mozilla/rust).

## Picking something interesting to do

To get an idea of where we're going, see the [[Note development roadmap]].

The way most people seem to get involved is by simply trying to write Rust code. Inevitably you will hit a bug or missing feature, at which point you may feel compelled to write a patch.

Another way to get involved is to look through the issue tracker for issues labeled with "E-easy", "I-enhancement", and/or "I-wishlist". Unassigned bugs are always fair game. Assigned bugs that don't seem to be getting worked on actively can be fair game, but always check with the bug owner first in that case. Also see [[Note development policy]].

Outstanding bugs or feature requests in Rust often have a corresponding test in the test suite that doesn't yet pass.  One good way to jump into Rust development is to look for files in the test/run-pass directory containing the string `xfail-test`.  Those tests all correspond to bugs that need to be fixed or features that someone needs to finish.

The source is also littered with hundreds of comments marked with 'FIXME' and 'TODO'. In Rust, FIXME comments come with an issue number; sometimes these refer to issues that may be resolved easily.

If in doubt, ask on IRC. Somebody will surely have a task that needs doing.

## Communicating

Join irc.mozilla.org #rust for real-time discussion of Rust development.  We try to remain on that channel during working hours in UTC-7 (US Pacific).

Join the [mailing list](https://mail.mozilla.org/listinfo/rust-dev) for longer conversations.

In both cases, please follow the conduct guidelines on the development policy in [[Note development policy]].