# Packaging

This page is summarizing some issues with the current (something pre-0.6) rust code which makes it difficult to package for downstream distributions - or even prevents it in some cases (e.g. because it doesn't follow the packaging policy of Fedora).

## Issues
### Hardcoded libdir
* [Issue 5223](https://github.com/mozilla/rust/issues/5223)
An hardcoded libdir is problematic when the downstream distribution uses arch specific libdirs e.g. lib for i686 and lib64 for x86_64

### Extensive usage of rpath
* [Issue 5219](https://github.com/mozilla/rust/issues/5219)
rpaths are problematic in several ways (see the links given in the issue).

### Patched LLVM - ARM only
* [Issue 4259](https://github.com/mozilla/rust/issues/4259)
At least for arm based builds rust is using a patched llvm.
It seems that a stock llvm can be used for x86 builds.But also see the next issue.

### `--llvm-root` switch doesn't work
* Issue
The `--llvm-root` switch should allow the usage of an external (existing) llvm, but this is currently not working.

### Tightly coupled with libuv
* [Issue 5563](https://github.com/mozilla/rust/issues/5563)
Rust is currently tighly coupled to libuv. Typically downstream distributions don't allow bundling of 3rdparty libs. A downstream packaging effort should package libuv in advance.
It might be a bit problematic to keep libuv and rust versions in sync during this early stage.

## Minor notes
* Bootstrap binary is required
