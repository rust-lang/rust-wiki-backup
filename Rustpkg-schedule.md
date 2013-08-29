This is a schedule for getting rustpkg to the point where we can build Rust with it, keeping in mind that like any package, rustpkg will never be complete. The goal is to reach the standard of quality implied by the issues up to and including those on Milestone 5 being complete.

Previous discussions about rustpkg planning:
* https://etherpad.mozilla.org/Meeting-weekly-2013-08-27

### "Ready for use" as per metabug:
 Due September 30

* [7075](https://github.com/mozilla/rust/issues/7075): rustpkg should use workcache
* [6408](https://github.com/mozilla/rust/issues/6408): Package IDs pointing into a subdirectory
* [8524](https://github.com/mozilla/rust/issues/8524): recursive dependencies
* [7402](https://github.com/mozilla/rust/issues/7402): Installing to RUST_PATH
* [8522](https://github.com/mozilla/rust/issues/8522): rustpkg should accept command line flags for rustc
* [6403](https://github.com/mozilla/rust/issues/6403): Teaching rustpkg how to build C libraries (at least partial support)

### Port Servo to rustpkg (concurrently with the following)

### Milestone 1: well-defined
  Due October 15

* [7447](https://github.com/mozilla/rust/issues/7447): Possible refinements to version handling
* [6746](https://github.com/mozilla/rust/issues/6746): Module file names should be resolved relatively to the referring module
* [6365](https://github.com/mozilla/rust/issues/6365): rustpkg usage messages need work

### Milestone 2: backwards-compatible
  Due October 25

* [8523](https://github.com/mozilla/rust/issues/8523): RFC: remove linkage attributes
* [6480](https://github.com/mozilla/rust/issues/6480): rustpkg should make locally cached files read-only

### Milestone 3: feature-complete
   Due December 16

* [8673](https://github.com/mozilla/rust/issues/8673): Specifying a revision in an `extern mod` directive can lead to duplication
* [8672](https://github.com/mozilla/rust/issues/8672): rustpkg should put build output in a target-specific subdirectory
* [8520](https://github.com/mozilla/rust/issues/8520): Let rustpkg find sources in the current working directory even if it's not a workspace
* [8405](https://github.com/mozilla/rust/issues/8405): rustpkg should support semantic versions
* [7401](https://github.com/mozilla/rust/issues/7401): Expand the set of commands that a package script can implement
* [7242](https://github.com/mozilla/rust/issues/7242): Finish implementing all rustpkg commands
* [6409](https://github.com/mozilla/rust/issues/6409): In rustpkg, expose the ability to clone a git repository
* [5012](https://github.com/mozilla/rust/issues/5012): `rustpkg`'s `-c` does not work.
* [2219](https://github.com/mozilla/rust/issues/2219): --attr flag
* [8405](https://github.com/mozilla/rust/issues/8405): Semantic versions
* [7240](https://github.com/mozilla/rust/issues/7240): Multi-crate packages

### Milestone 4: self-hosting
  Due January 31

* [5363](https://github.com/mozilla/rust/issues/5363): Build all rust libraries and binaries with rustpkg

### Milestone 5: production-ready
   Due February 21

* [8711](https://github.com/mozilla/rust/issues/8711): When searching the RUST_PATH, rustpkg ignores versions
* [8690](https://github.com/mozilla/rust/issues/8690): rustpkg test package_script_with_default_build makes bogus assumptions about the build directory layout
* [7879](https://github.com/mozilla/rust/issues/7879): rustpkg endlessly compiles dependencies
* [5219](https://github.com/mozilla/rust/issues/5219): Make rpath usage optional 
* [3346](https://github.com/mozilla/rust/issues/3346): crate name in log map is taken from name of output binary, not the crate name

### Milestone 6: far-future (everything else)

* [7243](https://github.com/mozilla/rust/issues/7243): Teach rustpkg how to unpack tarball packages and fetch via curl
* [6481](https://github.com/mozilla/rust/issues/6481): Add a lint command to rustpkg
* [6410](https://github.com/mozilla/rust/issues/6410): Write libgit2 bindings for Rust
* [6404](https://github.com/mozilla/rust/issues/6404): Teach rustpkg how to probe for C libraries and header files
* [6005](https://github.com/mozilla/rust/issues/6005): Rustpkg should know about docs
* [2805](https://github.com/mozilla/rust/issues/2805): Bindgen-based build tool that creates self-contained hybrid crates from foreign libraries
* [2124](https://github.com/mozilla/rust/issues/2124): Add a bindgen pass
* [1642](https://github.com/mozilla/rust/issues/1642): Continuous documentation server for rustpkg packages
* [1609](https://github.com/mozilla/rust/issues/1609): move rustpkg to its own repository
* [1453](https://github.com/mozilla/rust/issues/1453): Create a continuous integration server for building rustpkg packages
* [4019](https://github.com/mozilla/rust/issues/4019): Rewrite tests.mk in Rust