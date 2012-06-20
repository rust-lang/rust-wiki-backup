Incoming
--------

* The integration branch is `incoming`. Only the sheriff should push to `master`.
  * If you have push access and are not the sheriff today, please push to `incoming`, not `master`.
  * If you are filing pull requests, please file them against `incoming`, not `master`.
* The sheriff will pull the last green commit from `incoming` to `master` daily.
* The current sheriff will be listed in the IRC channel title.

General
-------

When submitting pull requests it is important to know your branch may be rebased, which will change its history and make your branch's history invalid.

One possible workflow that has worked for developers on the project is outlined below. This assumes that you have an 'origin' remote that represents your remote github repo, and a 'mozilla' remote that represents Mozilla's. To create the latter, you can execute:
```
$ git remote add mozilla git://github.com/mozilla/rust.git
```

1. Whenever you start working on anything new, create a new branch derived from mozilla's `master` branch:
```
$ git checkout master -b mybranch
```

2. While working, rebase your branch forwards regularly against the (daily) updates to `master`:
```
$ git checkout mybranch
$ git fetch mozilla
$ git rebase mozilla/master
```

> Sometimes there are conflicts during rebasing. If rebasing will cause some of your commits to not
> build or otherwise make less sense then don't do it (`git rebase --abort`).
> Instead finish the work on the current branch and submit the pull request without merging or rebasing.
> Do not repeatedly merge `master` into your branch. We will ask you to rewrite any pull request that
> contains unnecessary merge nodes.

3. When done, rebase one final time to the `incoming` branch of `mozilla/rust`, and push your work up to github:
```
$ git fetch mozilla
$ git rebase mozilla/incoming
$ <re-run local tests, make sure it still works>
$ git push origin mybranch
```

4. File a pull request against `mozilla/rust`. Target the pull request to the `incoming` integration branch in `mozilla/rust`, not `master`. Incoming changes target the `incoming` branch only. Once your pull request is filed, you can create a new branch and do something else.

5. After Mozilla merges your stuff to `incoming`, tests that it's green, and merges it over (semi-regularly) to `master`, it is integrated. Watch this process to ensure your change integrates. Address any test-breakage reported in the bots during integration.

6. Pull `master` into your local repo and verify that it contains your changes:
```
$ git checkout master
$ git pull
$ git log
  ... <look for your changes, confirm they arrived> ...
```

6. It is now safe to delete 'mybranch' from both local and remote repos:
```
$ git branch -D mybranch
$ git push origin :mybranch
```

## Git Submodules

In order to create a git submodule, follow this procedure:

1. First we have to prepare the submodule. If the project is hosted on http://github.com, fork it so we can apply changes. Otherwise check it out and push it into http://github.com/mozilla/ or wherever.

2. In the rust repository, create the submodule. Make sure that you specify a publicly accessible read-only url here, or else you may break the build for a non-rust developer:

```
$ git submodule add https://github.com/mozilla/libuv.git src/rt/libuv
```

3. Git will checkout the project into the subdirectory `src/rt/libuv`. We'll need to switch the origin to a writable repository so we can push to it.

```
$ cd src/rt/libuv
$ git remote set-url origin git@github.com:mozilla/libuv.git
```

4. I would suggest creating a branch to track the remote project, which we'll use to designate which revision we'll pin rust to. We'll also use this branch to track any rust customizations:

```
cd src/rt/libuv
$ git checkout -b rust ee599ec1141cc48f895de1f9d148033babdf9c2a
# make some changes to the submodule...
$ git push origin rust:refs/heads/rust
```

5. Finally, commit the submodule to rust. git should have automatically added/modified `.gitmodules` and `src/rt/libuv`:

```
$ git status
...
# Changes to be committed:
#   (use "git reset HEAD <file>..." to unstage)
#
#	modified:   .gitmodules
#	new file:   foobar
...
$ git commit -a
```

6. To be safe, check to make sure you can clone and build rust from the read-only repository:

```
$ git clone --recursive https://github.com/mozilla/rust.git
$ configure
$ make
```

## Updating a submodule

1. Make your changes to the submodule and commit them to the rust branch. Check `git status` to make sure there are not unknown files. If there are, this can confuse the supermodule into thinking that you still have uncommitted changes in the submodule. So if you do, either commit those files or add them to either the `.gitignore` or `.git/info/exclude` file.

```
$ cd src/rt/libuv
$ git checkout rust
# Make your changes...
$ git commit
$ git status
# On branch master
# Untracked files:
#   (use "git add <file>..." to include in what will be committed)
#
#	foo
nothing added to commit but untracked files present (use "git add" to track)
$ echo foo >> .gitignore
$ git add .gitignore
$ git commit
```

3. Commit the submodule changes into the supermodule.

```
$ cd $rustdir
$ git commit
```