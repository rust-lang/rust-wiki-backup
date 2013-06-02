This is Tim's attempt to come up with a schedule for finishing rustpkg, with a proposed deadline for each milestone.

0. Check in concrete test cases and/or English descriptions of how to test for this, for each item below

      ~~Due: 2013-04-17~~ Done ~~(pending pull request #5920)~~.

      1a. Clarify directory structure for package dirs (src/, lib/, build/, etc.) and make sure test cases reflect that. Also document this in English in a separate document.    
         ~~Due: 2013-04-19~~ Done ~~(pending pull request #5948)~~

      1b. Incorporate comments at https://github.com/mozilla/rust/pull/5948
          ~~Due: 2013-04-22~~ Done ~~(pending pull request #6008)~~
    
1. Fix package searching

   Look for packages in `./src` and not in `.` (as a stopgap before fully implementing `RUST_PATH`). Basically, respect the directory structure as specified in the draft manual, even if we're only searching in `.` and not the whole `RUST_PATH`. In my local copy of `src/librustpkg/testsuite/pass`, `rustpkg build hello-world` should work.
         
   ~~Due: 2013-04-22~~ Done ~~(pending pull request #6054)~~

1. Implement `install` command
      * test cases:
        * main.rs and lib.rs files end up in package source's install dirs (`lib` and `bin` by default)
      	* test/bench files end up in package source's build dir

   Pull request #6054 adds test cases

   ~~Due: 2013-04-26~~ Done ~~(pending pull request #6124)~~

   ~~Except for one hitch: need to infer the sysroot properly, at least for the tests to be able to run~~ Done

2. Full	pkgids (#5679)
      * test cases:
         * ~~deeply-nested subdirectory~~ Already working
      	 * ~~remote (github) URL~~ Done pending pull request #6418
      * Sub-tasks:
         * ~~Allow installing from URLs~~ Pull request #6418
         * ~~Ensure there's no longer any dependency on link name, URL, uuid~~
         * ~~Wipe out any traces of reverse-DNS-style paths if there are any~~
         * ~~Make sure, via unit tests, that "package IDs are relative-path-like" is enforced~~
         * ~~Make sure, via unit tests, that "package IDs have a single stem" is enforced~~

   ~~Due: 2013-05-12~~ Done ~~pending #6418~~

4. Finding external crates (#5681)
      * test cases:
          * a file with `extern mod foo` where foo lives in a different directory

   ~~Due: 2013-05-14~~ Done ~~pending #6807~~

3. Test	runner (#5683)

   Due: 2013-05-17

5. Get version from VCS (#5684)
      * test case:
          * use a subdirectory containing a git version, make sure the created executable or library has the right version number in the name

   Due: 2013-05-20

6. Implement `RUST_PATH` (#5682)
      * test cases:
          * searching in `./.rust`
      	  * install; make sure it installs to the first entry in `$RUST_PATH`
      	  * install from an entry in `RUST_PATH` that isn't the first one

   Due: 2013-05-22

6. Teach pkg.rs how to execute the default build logic (#6401)

   Due: 2013-05-24

6. Commands should work without an explicit package name (#6405)
      * build and install, to start with

   Due: 2013-05-25

7. Implement (or test) package database	and ability to list, add, remove packages
      * test case:
          * install several packages, make sure they appear in list
      	  * remove one package, make sure it doesn't appear in list and other packages do

   Due: 2013-05-27

8. `extern mod` should name a package ID
      * fix #6407 (change what's allowed in an `extern mod` directive)

8. Track dependencies between Rust packages
      * test case:
          * two packages, A and B; A depends on B with ```extern mod```; building A automatically builds B

   Due: 2013-05-29

8. Use workcache
      * test cases:
          * run `rustpkg build foo` twice, check timestamps to make sure foo isn't built again the second time
          * build foo, which depends on bar, from scratch; check timestamps to make sure foo was built after bar
          * build foo, which depends on bar, from scratch; change bar; rebuild foo; make sure foo was actually rebuilt (i.e. foo's timestamp is later than bar's timestamp)
          * use content hashing and not just datestamps (have tests where one, but not the other, changes)
      * generally, make sure ```rustpkg build``` "works like ```make```"
      * This depends on #4432

   Due: 2013-06-04

8. Extend pkg IDs to specify version explicitly
      * test case:
           * have two versions of package A, A 0.1 and A 0.2; installing A#0.1 doesn't install 0.2, does install 0.1
   
   At the same time, teach rustpkg about branches and tags (#6411)

   Due: 2013-06-07

8. Finish implementing all other commands
       * do, info, prefer, test, uninstall, unprefer

   Due: 2013-06-14

8. Expose the ability to clone a git repository (#6409)

8. Build C libraries (#6403 and #6404)

8. (wishlist) Track non-Rust dependencies
       * test case:
           * allow packages to declare their non-Rust dependencies; add a rustpkg command to print them all out