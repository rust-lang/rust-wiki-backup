[ccache]() is a tool which provides a cache for C/C++/Objective-C compiler operations.  We've found that it makes LLVM recompilation a bit easier to handle, and not everyone knows about it yet!

On Ubuntu, for example, you can use it without modifying any build scripts by simply installing the `ccache` package and putting links to it in your path like so:

```
apt-get install ccache
ln -s /usr/bin/ccache /usr/local/bin/gcc
ln -s /usr/bin/ccache /usr/local/bin/g++
ln -s /usr/bin/ccache /usr/local/bin/cc
```

Then you can verify that ccache is being used during builds:

```
ccache --show-stats
```