When submitting pull requests it is important to know that the rust developers make heavy use of rebasing. Branches that you submit for integration may be rebased, which will change their history and make your branch's history invalid.

One possible workflow that has worked for developers on the project is outlined below. This assumes that you have an 'origin' remote that represents your remote github repo, and a 'graydon' remote that represents graydon's.

1. Whenever you start working on anything new, create a new branch:
```
git checkout master -b mybranch
```

2. While working, rebase your branch forwards regularly:
```
git checkout master
git fetch graydon
git merge graydon/master
git rebase master mybranch
```


3. When done, push your work up to github:
```
git push origin mybranch
```

4. Make a pull request to Graydon.  In the meantime, you can create a new branch and do something else.  

5. After Graydon integrates your stuff, pull his master branch into your local repo:
```
git checkout master
git fetch graydon
git merge graydon/master
```

6. Verify that master contains your changes, then delete 'mybranch' from both local and remote repos:
```
git branch -D mybranch
git push origin :mybranch
```
