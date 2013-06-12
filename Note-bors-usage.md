(Adapted from an e-mail from Graydon to mailing list.)

There is an autolander running on mozilla/rust;
its name is bors (listed as a collaborator in the rust-push group).

It works by scanning pull reqs for r+ _on the commit_ (not the pull)
from one of the reviewers.
It then resets the 'auto' ref to incoming, attempts to merge the
reviewed commit into auto, and if that works, sends it for testing on
buildbot. If all tests come back green, it advances incoming to auto
(fast-forward only).

This _should_ mean that, insofar as all commits go through it, the tree
will always be green.

Not all commits go through it, because from time to time
we bypass it and break the tree ourselves; but we have made much use of it.

If you have any questions (or notice any glaring misbehavior or security
flaws) please let Graydon know and he'll turn it off. It is also .. somewhat
slow: github has a rate-limited API so it only cycles one step every 2
minutes. Which is plenty fast enough to integrate changes, but not
instant-feedback fast.