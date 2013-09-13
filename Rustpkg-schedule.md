This is a schedule for getting rustpkg to the point where we can build Servo with it, keeping in mind that like any package, rustpkg will never be complete. The goal is to reach the standard of quality implied by the issues up to and including those on Milestone 5 being complete.

Previous discussions about rustpkg planning:
* https://etherpad.mozilla.org/Meeting-weekly-2013-08-27

## First 10 servo crates: 9/11/2013
 * ~~[#7075](https://github.com/mozilla/rust/issues/7075) - workcache~~
 * ~~[#8522](https://github.com/mozilla/rust/issues/8522) - rustc flags~~ - 9/11/2013

## Build all of servo: 10/1/2013
 * ~~[#6408](https://github.com/mozilla/rust/issues/6408) - pkgids with subdirs~~ - 9/16/2013 (medium)                 
 * ~~[#8524](https://github.com/mozilla/rust/issues/8524) - recursive dependencies~~ - 9/18/2013 (easy)
 * ~~[#7402](https://github.com/mozilla/rust/issues/7402) - install to RUST_PATH~~ Done pending review of pull request [#9147](https://github.com/mozilla/rust/pull/9147) - 9/19/2013 (easy)
 * Tim on PTO, 9/20             
 * [#7879](https://github.com/mozilla/rust/issues/7879) - infinite loop compiling dependencies - 9/24/2013 (hard)
 * [#6403](https://github.com/mozilla/rust/issues/6403) - building c libs - 9/30/2013 (medium)                     
 * [#8672](https://github.com/mozilla/rust/issues/8672) - target specific output directory - 10/1/2013 (easy) 

## Community adoption I: 10/29/2013
 * [#6480](https://github.com/mozilla/rust/issues/6480) - make locally-cached files read only - 10/3/2013 (easy)
 * Mozilla Summit, 10/4-10/7
 * [#8520](https://github.com/mozilla/rust/issues/8520) - find sources in $CWD - 10/10/2013 (medium)
 * [#7240](https://github.com/mozilla/rust/issues/7240) - multi-crate packages - 10/17/2013 (hard)
 * [#8711](https://github.com/mozilla/rust/issues/8711) - correctly deal with versions of installed pkgs - 10/23/2013 (medium)
 * [#9045](https://github.com/mozilla/rust/issues/9045) - rustpkg init (graydon's .rust_workspace suggestion) - 10/29/2013 (medium)

## Community adoption II: 12/5/2013
 * [#7401](https://github.com/mozilla/rust/issues/7401) - support custom commands - 11/6/2013 (hard)
 * [#6365](https://github.com/mozilla/rust/issues/6365) - usage messages - 11/7/2013 (easy)
 * [#8892](https://github.com/mozilla/rust/issues/8892) - suppress installation of some crate files (the "glfw/examples" problem) - 11/12/2013 (medium)
 * [#8405](https://github.com/mozilla/rust/issues/8405) - semantic versions - 11/20/2013 (hard)
 * [#7242](https://github.com/mozilla/rust/issues/7242) - finish all commands - 11/27/2013 (particularly `do` and `info`) (hard)
 * Thanksgiving holiday 11/28-11/29
 * [#6005](https://github.com/mozilla/rust/issues/6005) - rustdoc integration - 12/5/2013 (hard)

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
 * #8711 - version matching
 * #7744 - error handling with bad package IDs
 * #2219 - --attr flag
 * #8871 - where to install remote packages
 * #8952 - executable naming
 * #7243 - unpacking tarball packages, fetching via curl
 * #9003 - `rustpkg test` cleanup