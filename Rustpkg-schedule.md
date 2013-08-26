"Ready for use" as per metabug:
Due September 30

* 7075: rustpkg should use workcache
* 6408: Package IDs pointing into a subdirectory
* 8524: recursive dependencies
* 7402: Installing to RUST_PATH
* 8405: Semantic versions
* 7240: Multi-crate packages

Port Servo to rustpkg (concurrently with the following)

Milestone 1: well-defined
Due October 15

* 7447: Possible refinements to version handling
* 6746: Module file names should be resolved relatively to the referring module
* 6365: rustpkg usage messages need work

Milestone 2: backwards-compatible
Due October 25

* 8523: RFC: remove linkage attributes
* 6480: rustpkg should make locally cached files read-only

Milestone 3: feature-complete
Due December 16

* 8673: Specifying a revision in an `extern mod` directive can lead to duplication
* 8672: rustpkg should put build output in a target-specific subdirectory
* 8522: rustpkg should accept command line flags for rustc
* 8520: Let rustpkg find sources in the current working directory even if it's not a workspace
* 8405: rustpkg should support semantic versions
* 7401: Expand the set of commands that a package script can implement
* 7242: Finish implementing all rustpkg commands
* 6409: In rustpkg, expose the ability to clone a git repository
* 6403: Teaching rustpkg how to build C libraries
* 5012: `rustpkg`'s `-c` does not work.
* 2219: --attr flag

Milestone 4: self-hosting
Due January 15

* 5363: Build all rust libraries and binaries with rustpkg
* 4019: Rewrite tests.mk in Rust

Milestone 5: production-ready
Due February 7

* 8711: When searching the RUST_PATH, rustpkg ignores versions
* 8690: rustpkg test package_script_with_default_build makes bogus assumptions about the build directory layout
* 7879: rustpkg endlessly compiles dependencies
* 5219: Make rpath usage optional 
* 3346: crate name in log map is taken from name of output binary, not the crate name

Milestone 6: far-future (everything else)
* 7243: Teach rustpkg how to unpack tarball packages and fetch via curl
* 6481: Add a lint command to rustpkg
* 6410: Write libgit2 bindings for Rust
* 6404: Teach rustpkg how to probe for C libraries and header files
* 6005: Rustpkg should know about docs
* 2805: Bindgen-based build tool that creates self-contained hybrid crates from foreign libraries
* 2124: Add a bindgen pass
* 1642: Continuous documentation server for rustpkg packages
* 1609: move rustpkg to its own repository
* 1453: Create a continuous integration server for building rustpkg packages