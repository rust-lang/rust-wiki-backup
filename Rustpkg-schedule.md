This is a schedule for getting rustpkg to the point where we can build Servo with it, keeping in mind that like any package, rustpkg will never be complete. The goal is to reach the standard of quality implied by the issues up to and including those on Milestone 5 being complete.

Previous discussions about rustpkg planning:
* https://etherpad.mozilla.org/Meeting-weekly-2013-08-27

## First 10 servo crates: 9/11/2013
 * ~~[#7075](https://github.com/mozilla/rust/issues/7075) - workcache~~ Ready to land (pending minor cleanup)
 * [#8522](https://github.com/mozilla/rust/issues/8522) - rustc flags - 9/11/2013

## Build all of servo: 9/30/2013
 * [#6408](https://github.com/mozilla/rust/issues/6408) - pkgids with subdirs - 9/14/2013                 
 * [#8524](https://github.com/mozilla/rust/issues/8524) - recursive dependencies - 9/19/2013              
 * [#6403](https://github.com/mozilla/rust/issues/6403) - building c libs - 9/26/2013                     
 * [#8672](https://github.com/mozilla/rust/issues/8672) - target specific output directory - 9/30/2013   

## Community adoption: 12/13/2013
 * [#6480](https://github.com/mozilla/rust/issues/6480) - make locally-cached files read only - 10/9/2013
 * [#8520](https://github.com/mozilla/rust/issues/8520) - find sources in $CWD - 10/16/2013
 * [#7240](https://github.com/mozilla/rust/issues/7240) - multi-crate packages - 10/22/2013
 * [#8711](https://github.com/mozilla/rust/issues/8711) - correctly deal with versions of installed pkgs - 10/29/2013
 * [#9045](https://github.com/mozilla/rust/issues/9045) - rustpkg init (graydon's .rust_workspace suggestion) - 11/6/2013
 * [#7401](https://github.com/mozilla/rust/issues/7401) - support custom commands - 11/14/2013
 * [#6365](https://github.com/mozilla/rust/issues/6365) - usage messages - 11/16/2013
 * [#8405](https://github.com/mozilla/rust/issues/8405) - semantic versions - 11/25/2013
 * [#7242](https://github.com/mozilla/rust/issues/7242) - finish all commands - 12/4/2013
 * [#6005](https://github.com/mozilla/rust/issues/6005) - rustdoc integration - 12/13/2013

## Everything else (To be scheduled once community adoption is complete or close to complete)
 * #7447 - version improvements
 * #3346 - crate name in log map
 * #6409 - API for `git clone`
 * #6410 - libgit2 bindings
 * #1453 - continuous integration server
 * #1642 - continuous documentation server
 * #5219 - rpath
 * #2124 - bindgen
 * #8523 - forbid linkage attributes for rustpkg crates
 * #8673 - extern mod duplication
 * #6481 - lint command
 * #7879 - infinite loop compiling dependencies
 * #8711 - version matching
 * #7744 - error handling with bad package IDs
 * #2219 - --attr flag
 * #8871 - where to install remote packages
 * #8892 - suppress installation of some crate files
 * #8952 - executable naming
 * #7243 - unpacking tarball packages, fetching via curl
 * #9003 - `rustpkg test` cleanup