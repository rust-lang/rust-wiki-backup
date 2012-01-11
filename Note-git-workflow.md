When submitting pull requests it is important to know that the rust developers make heavy use of rebasing. Branches that you submit for integration may be rebased, which will change their history and make your branch's history invalid.

One possible workflow that has worked for developers on the project is outlined below. This assumes that you have an 'origin' remote that represents your remote github repo, and a 'mozilla' remote that represents Mozilla's. To create the latter, you can execute:
```
$ git remote add mozilla git://github.com/mozilla/rust.git
```

1. Whenever you start working on anything new, create a new branch:
```
$ git checkout master -b mybranch
```

2. While working, rebase your branch forwards regularly:
```
$ git checkout master
$ git fetch mozilla
$ git merge mozilla/master
$ git rebase master mybranch
```


3. When done, push your work up to github:
```
$ git push origin mybranch
```

4. Make a pull request to Mozilla.  In the meantime, you can create a new branch and do something else.

5. After Mozilla integrates your stuff, pull that master branch into your local repo:
```
$ git checkout master
$ git fetch mozilla
$ git merge mozilla/master
```

6. Verify that master contains your changes, then delete 'mybranch' from both local and remote repos:
```
$ git branch -D mybranch
$ git push origin :mybranch
```

## Git Submodules

In order to create a git submodule, follow this procedure:

1. First we have to prepare the submodule. If the project is hosted on http://github.com, fork it into http://github.com/graydon. Otherwise check it out and push it into http://github.com/graydon.

2. In the rust repository, create the submodule. Make sure that you specify a publicly accessible read-only url here, or else you may break the build for a non-rust developer:

```
$ git submodule add https://github.com/graydon/libuv.git src/rt/libuv
```

3. Git will checkout the project into the subdirectory `src/rt/libuv`. We'll need to switch the origin to a writable repository so we can push to it.

```
$ cd src/rt/libuv
$ git remote set-url origin git@github.com:graydon/libuv.git
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