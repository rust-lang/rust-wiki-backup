This is Tim's attempt to come up with a schedule for finishing rustpkg, with a proposed deadline for each milestone.

0. Check in concrete test cases and/or English descriptions of how to test for this, for each item below

      ~~Due: 2013-04-17~~ Done ~~(pending pull request #5920)~~.

      0a. Clarify directory structure for package dirs (src/, lib/, build/, etc.) and make sure test cases reflect that. Also document this in English in a separate document.
         Due: 2013-04-19

1. Implement 'install' command
      * test cases:
        * main/lib files end up in specified install dir
      	* test/bench files stay in build subdir

   Due: 2013-04-22

2. Full	pkgids (#5679)
      * test cases:
         * deeply-nested subdirectory
      	 * remote (github) URL

   Due: 2013-04-24

3. Test	runner (#5683)

   Due: 2013-04-30

4. Finding external crates (#5681)
      * test cases:
          * a file with "extern mod foo" where foo lives in a different directory

   Due: 2013-05-03

5. Get version from VCS (#5684)
      * test case:
          * use a subdirectory containing a git version, make sure the created executable or library has the right version number in the name

   Due: 2013-05-07

6. Implement RUST_PATH (#5682)
      * test cases:
          * searching in .rust
      	  * install; make sure it installs to the first entry in $RUST_PATH
      	  * install from an entry in RUST_PATH that isn't the first one

   Due: 2013-05-10

7. Implement (or test) package database	and ability to list, add, remove packages
      * test case:
          * install several packages, make sure they appear in list
      	  * remove one package, make sure it doesn't appear in list and other packages do

   Due: 2013-05-17

8. Track dependencies between Rust packages
      * test case:
          * two packages, A and B; A depends on B with ```extern mod```; building A automatically builds B

   Due: 2013-05-28

8. Use workcache
      * test cases:
          * run "rustpkg build foo" twice, check timestamps to make sure foo isn't built again the second time
          * build foo, which depends on bar, from scratch; check timestamps to make sure foo was built after bar
          * build foo, which depends on bar, from scratch; change bar; rebuild foo; make sure foo was actually rebuilt (i.e. foo's timestamp is later than bar's timestamp)
          * use content hashing and not just datestamps (have tests where one, but not the other, changes)
      * generally, make sure ```rustpkg build``` "works like ```make```"
      * This depends on #4432

   Due: 2013-06-04

8. Extend pkg IDs to specify version explicitly
      * test case:
           * have two versions of package A, A 0.1 and A 0.2; installing A#0.1 doesn't install 0.2, does install 0.1

   Due: 2013-06-07

8. Finish implementing all other commands
       * do, info, prefer, test, uninstall, unprefer

   Due: 2013-06-14

8. (wishlist) Track non-Rust dependencies
       * test case:
           * allow packages to declare their non-Rust dependencies; add a rustpkg command to print them all out