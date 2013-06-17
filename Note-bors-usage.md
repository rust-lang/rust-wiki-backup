(Adapted from e-mails from Graydon to mailing list.)

There is an auto-merge robot running on mozilla/rust; its name is "bors";
it has a github account and is listed as a collaborator in the `rust-push` group.

Its status URL is: http://buildbot.rust-lang.org/bors/bors.html

It works by scanning pull requests for r+ _on the commit_ (*not* the pull request)
from one of the reviewers.

For the most part no knowledge of bors is required of people submitting pull
requests.  When a reviewer signs off on one of your commits by writing "r+" 
in a comment on the final commit of a pull request, bors will automatically:

  - Form a new merge node based on your pull + incoming
  - Wait for results from that to come back from buildbot
  - If successful, advance incoming to that node
  - Otherwise report how the change failed

It then resets the 'auto' ref to incoming, attempts to merge the
reviewed commit into auto, and if that works, sends it for testing on
buildbot. If all tests come back green, it advances incoming to auto
(fast-forward only).  Rebased pull requests require
re-approval.

This _should_ mean that, insofar as all commits go through bors, the tree
will always be green.

There is a certain degree of chatter from bors as it steps through its automation.
It makes comments, such as in this pull request:

  https://github.com/mozilla/rust/pull/4832

This is just automation of a somewhat tedious merge-making and
testsuite-watching behavior that we were previously doing manually (and
spending a lot of time at, and occasionally skipping steps of).
Automating it frees up time, lets the robot do useful work overnight
when reviewers are asleep, and ensures that we never skip running tests
"optimistically". Incoming only ever advances to merge nodes that are
_exactly_ the bits that tested OK.

It retries if you move incoming while it's working, so tends to behave
"mostly gracefully" with respect to disruption around it. If it
misbehaves, it can be provoked to retry a commit with a comment from a
reviewer saying "@bors: retry" (on that same commit as the original "r+", as
opposed to the comment-trail inlined the Pull Request itself).
It ignores comments by non-reviewers. It
also does not merge "updates" to a pull request. It considers comments on
commits only, not pull requests; if you update a pull request to contain new
commits, they need to be reviewed anew.

Not all commits go through it, because from time to time
we bypass it and break the tree ourselves; but we have made much use of it.

If you have feature requests or want it to behave differently, please
let us know.  We are conducting a security review with some
mozilla folks and will be open-sourcing it soon for others who want to apply
similar behavior to their own repositories. It's a very small, simple
script.

It is somewhat slow: github has a rate-limited API so it only cycles one step every 2
minutes. Which is plenty fast enough to integrate changes, but not
instant-feedback fast.