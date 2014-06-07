There is an auto-merge robot running on mozilla/rust; its name is "bors";
it has a github account and is listed as a collaborator in the `rust-push` group.
Its source is available [on GitHub](https://github.com/graydon/bors).

Its status URL is: http://buildbot.rust-lang.org/bors/bors.html

It works by scanning pull requests for r+ _on the commit_ (*not* the pull request)
from one of the reviewers. It also accepts the following input:

* `r=name[,name...]` to specify that the given person(s) should be marked as the reviewer(s), rather than whoever left the comment
* `p=number` to specify the priority that the PR should be tested. Higher priority is tested first. Ties are resolved by date the PR was opened.

For the most part no knowledge of bors is required of people submitting pull
requests.  When a reviewer signs off on one of your commits by writing "r+" 
in a comment on the final commit of a pull request, bors will automatically:

  - Reset the `auto` branch to `master`
  - Attempt to merge your pull request into `auto` --- if this fails, your request will be marked as stale, meaning you will need to rebase it.
  - Wait for buildbot to test `auto`
  - If successful, fast-forward `master` to `auto`
  - Otherwise report how the change failed

Rebased pull requests require re-approval.

There is a certain degree of chatter from bors as it steps through its automation.
It makes comments, such as in this pull request:

  https://github.com/mozilla/rust/pull/4832

It retries if you move incoming while it's working, so tends to behave
"mostly gracefully" with respect to disruption around it. If it
misbehaves, it can be provoked to retry a commit with a comment from a
reviewer saying "@bors: retry" (on that same commit as the original "r+", as
opposed to the comment-trail inlined the Pull Request itself).
It ignores comments by non-reviewers. It
also does not merge "updates" to a pull request. It considers comments on
commits only, not pull requests; if you update a pull request to contain new
commits, they need to be reviewed anew.