# Overview

The compiler (`rustc`) is self-hosting. This means that while you develop, you "start" from a pre-built binary we serve from an archive server, and build new copies of the same program (from the same source) in your workspace.

Self-hosting involves "stages". There are 4 stages to know about:

* `stage0` is the stage you fetched from a snapshot server. The binaries in here correspond to versions from "recently" that were built on one of our development machines and uploaded to our archive server. The build system will download the snapshots into your workspace when building, in order to get you started, and pick up new snapshots as we make them.

* `stage1` is your workspace compiled with `stage0`. It will contain any changes you made, but it will not yet have been *built with* those changes. It was built with a snapshot compiler.

* `stage2` is your workspace compiled with `stage1`. It was built with your changes.

* `stage3` is your workspace compiled with `stage2`; it should equal stage2, and is built only as a check to ensure that your change "converges" to a fixpoint where multiple compilations of self result in the same binary image.

The stage0 snapshots originated from a bootstrap compiler that has since been retired. We're running on all snapshots now.

Occasionally we will make new snapshots. This may happen just to get a performance or code-quality improvement that is present on trunk, or it may have to do with wanting to use a feature in the code that is present on trunk. In any case the snapshots may need updating.

Snapshots are made on a self-serve basis by users with write access to the repository. Two kinds of snapshots can be made.

* `stage3` snapshots are _what you make most of the time_, when adding a backwards-compatible feature or optimization. That is, any time you introduce a change where the compiler doesn't have to be changed in order to continue to compile through to stage3 with that change.

* `stage1` snapshots _may_ be necessary, in passing, to make "transitional" compilers you use during a non-backwards-compatible change. These are rare and require special treatment, below. _Try to avoid_ the need to make `stage1` snapshots as they're a bit delicate; if you can get away with adding a parallel feature at some point, then removing support for its old uses once they've died out, that's preferable.

# Procedures

All snapshots should be possible in a "self-serve" fashion now, for users with direct write access to the development repo. Follow the instructions below. Unfortunately they're still a bit heavy on manual work by the user, but the upside is you can learn to do it all yourself, without relying on anyone else being online. We'll make this more automatic as time goes by.

If you have _any questions_ about these procedures, please drop by IRC and ask in person, and/or send mail to the mailing list asking for help. You can make a bit of a mess if you make a mistake during migration, so it's best to be sure what you're doing.

## READ THIS PART

While you're making snapshots, you'll be pushing to non-`master` branches until your work is done and you need to integrate it back to `master`. Other people may be pushing to `master` in the meantime. It is essential that you _never_ rebase your snapshot-making commits when finishing your work and pushing to `master`. Use `git merge` if anyone else races with you in the meantime. Snapshots are identified by revision ID and those will change if you rebase.

You should probably also not race with other users who are themselves making snapshots; in theory it will work but only if the tinderboxes don't overlook your pushes. Since the tinderboxes are inexact (they just pull "the most recent change on a branch") it's possible no snapshot will get built for you, which will be sad.

## Stage3 (backwards-compatible) snapshots

* Make your change in your workspace. Check that it builds and passes `make check`.

* Commit your change but do _not_ push to `master`; instead push to `snap-stage3`. You do this using the `local:remote` convention in git, pushing _your_ `master` branch to _Graydon's_ `snap-stage3` branch. Assuming you have a remote configured for graydon, that is, you run:
```
$ git push -f graydon master:snap-stage3
```

* Wait for the tinderboxes to cycle. They will get around to building your commit. Look for a green box to show up with your name and `snap-stage3` (assuming you're not racing with someone else making a snapshot).

* Open the tabs corresponding to the short build logs for the snapshots made on all 3 platforms -- you get that by clicking the blue `L` link on the green box -- and scroll to the bottom of the logs. Look for the line showing the upload to S3, which looks like so:
```
s3cmd put -P rust-stage0-2011-05-13-0d32ff7-linux-i386-4adfe572211e609bf8faeb327ffabc4bb6bc3a1e.tar.bz2 ...
```

* Open the `src/snapshots.txt` file and make a new 'S' entry at the _top_ of the file. It should look like this:
```
     S 2011-05-13 0d32ff7
       linux-i386 4adfe572211e609bf8faeb327ffabc4bb6bc3a1e
       macos-i386 bc7ee4d146ef6e0236afbd7cc4a9241582fd2952
       winnt-i386 5d3279a2dd0e3179b0e982432d63d79f87cac221
```
    These are examples; you should use the values from the lines you pulled out of the logs. The thing to notice is that the first part `S 2011-05-13 0d32ff7` names the git revision you pushed, and the subsequent 3 indented lines name the per-platform snapshots and give their sha1 values, that you pulled out of the builder logs. Be sure to actually add all 3 lines, otherwise you'll be leaving some platform out in the cold. That's not cool.

* Save `src/snapshots.txt` and make check in your workspace. Make sure everything's cool. You are now building with your new snapshot locally.

* If that worked, commit and push to `master`. Now everyone will be using your snapshot. If there have been changes on `master` in the meantime, _you must_ merge with them before pushing, _not_ rebase onto them.

## Stage1 (backwards-incompatible) snapshots

* Read through the bit above about making backwards-compatible changes. Make sure you understand that, since we'll just describe a variation of that procedure here.

* For the sake of example, let's say you're making a change to the rust grammar to change `fn` to `func`. You have to do this in two steps: 
    * make a compiler that parses `func` but not `fn`, and make a `snap-stage1` compiler.
    * edit the compiler source, changing all occurrences of `fn` to `func`, then _use_ that `snap-stage1` compiler to make a `snap-stage3` compiler.

* The tricky part is sequencing these changes.
    * Make your first change. That is, change the parser to recognize `func` but not `fn`. Then add a line at the start of `src/snapshots.txt` consisting just of the letter `T`. The presence of `T` in the first column of the first line of that file inhibits building beyond stage1; it tells the build system that you're in the middle of a transition. Be sure it builds to stage1 with `make`.
    * Commit your changes (including the `T` line in `src/snapshots.txt`) and push to Graydon's `snap-stage1`. That is, push with:
```
    git push -f graydon master:snap-stage1
```
    * Wait for the tinderboxes to cycle as above and copy down the filenames uploaded to S3 in the end of the build logs. Modify the entry in `src/snapshots.txt` to complete the `T` entry. It should look like an `S` entry, as above, but with `T` in place of `S`. So it should look like this:
```
     T 2011-05-13 0d32ff7
       linux-i386 4adfe572211e609bf8faeb327ffabc4bb6bc3a1e
       macos-i386 bc7ee4d146ef6e0236afbd7cc4a9241582fd2952
       winnt-i386 5d3279a2dd0e3179b0e982432d63d79f87cac221
```
    * Do not commit that yet.
    * Make your change to the compiler sources now. That is, change `fn` to `func` everywhere, in our example.
    * Commit this change along with the completed `T` snapshot registration in `src/snapshots.txt`.
    * Push to Graydon's `snap-stage3`
    * Wait for the tinderboxes to cycle as above and make a final entry in `src/snapshots.txt`, this time making an `S` entry as you would do for a compatible change.
    * Commit this final snapshot registration, and push to `master`. If there have been changes on `master` in the meantime, _you must_ merge with them before pushing, _not_ rebase onto them.
