Whenever you start working on anything new, create a new branch:
```
git checkout master -b myproject
```
When done, push your stuff up to github:
```
git push origin myproject
```
Then make a pull request to Graydon.  In the meantime, you can create a new branch and do something else.  After Graydon integrates your stuff, do:
```
git checkout master; git fetch graydon; git merge graydon/master
```
which will switch your local workspace to 'master', and bring it up to
date with graydon.

Then you should delete your 'myproject' branch that's no longer
needed.  The following throws away 'myproject' and deletes it from the
server.
```
git branch -D myproject; git push origin :myproject
```


