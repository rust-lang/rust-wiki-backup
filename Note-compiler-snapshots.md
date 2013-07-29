The compiler (`rustc`) is self-hosting. This means that while you develop, you "start" from a pre-built binary we serve from an archive server, and build new copies of the same program (from the same source) in your workspace.

Self-hosting involves "stages". There are 4 stages to know about:

* `stage0` is the stage you fetched from a snapshot server. The binaries in here correspond to versions from "recently" that were built on one of our development machines and uploaded to our archive server. The build system will download the snapshots into your workspace when building, in order to get you started, and pick up new snapshots as we make them.

* `stage1` is your workspace compiled with `stage0`. It will contain any changes you made, but it will not yet have been *built with* those changes. It was built with a snapshot compiler.

* `stage2` is your workspace compiled with `stage1`. It was built with your changes.

* `stage3` is your workspace compiled with `stage2`; it should equal stage2, and is built only as a check to ensure that your change "converges" to a fixpoint where multiple compilations of self will result in the same binary image.

The stage0 snapshots originated from a bootstrap compiler that has since been retired. We're running on all snapshots now.

Occasionally we will make new snapshots. This may happen just to get a performance or code-quality improvement that is present on trunk, or it may have to do with wanting to use a feature in the code that is present on trunk. In any case the snapshots may need updating.

Snapshots are made on a self-serve basis by users with write access to the repository.

# Procedures

All snapshots should be possible in a "self-serve" fashion now, for users with direct write access to the development repo. Follow the instructions below. Unfortunately they're still a bit heavy on manual work by the user, but the upside is you can learn to do it all yourself, without relying on anyone else being online. We'll make this more automatic as time goes by.

If you have _any questions_ about these procedures, please drop by IRC and ask in person, and/or send mail to the mailing list asking for help. You can make a bit of a mess if you make a mistake during migration, so it's best to be sure what you're doing.

## READ THIS PART

While you're making snapshots, you'll be pushing to non-`master` branches until your work is done and you need to integrate it back to `master`. [[Bors|Note-bors-usage]] or other people may be pushing to `master` in the meantime. It is essential that you _never_ rebase your snapshot-making commits when finishing your work and pushing to `master`. Use `git merge` if anyone else races with you in the meantime. Snapshots are identified by revision ID and those will change if you rebase. To avoid confusion, in general you should only create snapshots from commits that are already on master.

You should probably also not race with other users who are themselves making snapshots; in theory it will work but only if the buildbots don't overlook your pushes. Since the buildbots are inexact (they just pull "the most recent change on a branch") it's possible no snapshot will get built for you, which will be sad.

## Stage3 snapshots

* Push the commit you want to snapshot to the `snap-stage3` branch. You do this using the `local:remote` convention in git, pushing _your_ `master` branch to _Mozilla's_ `snap-stage3` branch. Assuming you have a remote configured for mozilla, that is, you run:
```
$ git push -f mozilla master:snap-stage3
```

* Wait for the [buildbots](http://buildbot.rust-lang.org/builders) to cycle. They will get around to building your commit. All of the `snap-X` builders will need to build green before the snapshot can be completed.

* Open the build results of each of the snapshot builders. Do this by clicking on the most recent build number for each snapshot builder. Each builder logs a number of build steps and will have one or more lines that read something like

```
uploading rust-stage0-2013-07-29-63c9b11-freebsd-x86_64-e2aa76ac1d3edd5385cb572b47f34ecfa8c611d1.tar.bz2
```

* Open the `src/snapshots.txt` file and make a new 'S' entry at the _top_ of the file. It should look something like this:

```
     S 2011-05-13 0d32ff7
       linux-i386 4adfe572211e609bf8faeb327ffabc4bb6bc3a1e
       macos-i386 bc7ee4d146ef6e0236afbd7cc4a9241582fd2952
       winnt-i386 5d3279a2dd0e3179b0e982432d63d79f87cac221
```

These are examples; you should use the values from the lines you pulled out of the logs. The thing to notice is that the first part `S 2011-05-13 0d32ff7` names the git revision you pushed, and the subsequent 3 indented lines name the per-platform snapshots and give their sha1 values, that you pulled out of the builder logs. Be sure to actually add the same number of lines as were included in the previous snapshot, otherwise you'll be leaving some platform out in the cold. That's not cool.

* Save `src/snapshots.txt` and make check in your workspace. Make sure everything's cool. You are now building with your new snapshot locally.

* Do a build locally, push to your branch and open a normal pull request. Bors will integrate it and then everybody will be using your snapshot.

## Local snapshots

Sometimes you will want to make a snapshot just on your machine for testing. Executing ```make snap-stage3``` in your Rust build directory should leave behind a ```.bz2``` file. To rebuild Rust with the new snapshot, execute:

```
CFG_SRC_DIR=$HOME/rust ../src/etc/get-snapshot.py x86_64-apple-darwin \
     rust-stage0-2012-11-03-444a16a-macos-x86_64-ed4b8355bfb1bcea6216bac585053a67e05df8a2.tar.bz2
```

modifying the value of ```CFG_SRC_DIR``` appropriately; using the correct host triple for your machine; and substituting whatever the name of the ```.bz2``` file that was just created. This will unzip the new snapshot and install it. Then you can run ```make check``` as normal.