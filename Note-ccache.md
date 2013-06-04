[ccache](http://ccache.samba.org/) is a tool which provides a cache for C/C++/Objective-C compiler operations.  We've found that it makes LLVM recompilation a bit easier to handle, and not everyone knows about it yet!

After installing the `ccache` package using your preferred package manager (i.e. `apt-get`, `yum`, `pacman`), you need to prioritize `ccache` so that it intercepts calls to the compiler. There are three methods for doing so.

The first method is to pass `--enable-ccache` when you run `configure`.

The second method is symlink `ccache` to a higher priority directory in $PATH, e.g. `/usr/local/bin`.

```
$ sudo ln -s /usr/bin/ccache /usr/local/bin/gcc
$ sudo ln -s /usr/bin/ccache /usr/local/bin/g++
$ sudo ln -s /usr/bin/ccache /usr/local/bin/cc
```

The third method is to edit `~/.bashrc` and add ccache's bin directory into the beginning of your $PATH. Please note that depending on your distro, the binaries will be in one of two different locations:

```
export PATH=/usr/lib/ccache/bin:/usr/lib/ccache:${PATH}
```

Then you can verify that ccache is being used during builds:

```
ccache --show-stats
```

There are also some [concerns](http://petereisentraut.blogspot.com/2011/09/ccache-and-clang-part-2.html) using `ccache` with `clang` that should be noted.