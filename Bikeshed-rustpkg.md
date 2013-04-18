This is Tim's attempt to come up with a schedule for finishing rustpkg, with a proposed deadline for each milestone.

0. Check in concrete test cases and/or English descriptions of how to test for this, for each item below

1. Implement 'install' command
      * test cases:
        * main/lib files end up in specified install dir
      	* test/bench files stay in build subdir

2. Full	pkgids (#5679)
      * test cases:
         * deeply-nested subdirectory
      	 * remote (github) URL

3. Test	runner (#5683)

4. Finding external crates (#5681)
      * test cases:
          * a file with "extern mod foo" where foo lives in a different directory

5. Get version from VCS (#5684)
      * test case:
          * use a subdirectory containing a git version, make sure the created executable or library has the right version number in the name

6. Implement RUST_PATH (#5682)
      * test cases:
          * searching in .rust
      	  * install; make sure it installs to the first entry in $RUST_PATH
      	  * install from an entry in RUST_PATH that isn't the first one

7. Implement (or test) package database	and ability to list, add, remove packages
      * test case:
          * install several packages, make sure they appear in list
      	  * remove one package, make sure it doesn't appear in list and other packages do

8. Use workcache
      * test case:
          * run "rustpkg build foo" twice, check timestamps to make sure foo isn't built again the second time
      * This depends on #4432

8. Track dependencies between Rust packages
      * test case:
          * two packages, A and B; A depends on B with ```extern mod```; building A automatically builds B

8. Extend pkg IDs to specify version explicitly
      * test case:
           * have two versions of package A, A 0.1 and A 0.2; installing A#0.1 doesn't install 0.2, does install 0.1

8. Finish implementing all other commands
       * do, info, prefer, test, uninstall, unprefer

8. (wishlist) Track non-Rust dependencies
       * test case:
           * allow packages to declare their non-Rust dependencies; add a rustpkg command to print them all out