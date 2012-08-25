Development policy for Rust.

## Contributions and commit access

If you are already have <a href="http://www.mozilla.org/hacking/commit-access-policy/">Level 1 Access</a> to commit within the Mozilla project you may commit directly to this repository. Email one of the existing authors and ask for commit access if you want to be contributing quickly and regularly this way.

More likely is a fork and pull request, for which you will need a Github account; if you're unclear on this process, see Github's documentation on [forking](https://help.github.com/articles/fork-a-repo) and [pull requests](https://help.github.com/articles/using-pull-requests). Pull requests should be targeted at Rust's `incoming` branch (note that by default Github will aim them at the `master` branch) -- see "Changing The Commit Range and Destination Repository" in Github's documentation on [pull requests](https://help.github.com/articles/using-pull-requests). Before pushing to your Github repo and issuing the pull request, please do two things:

1. [Rebase](http://git-scm.com/book/en/Git-Branching-Rebasing) your local changes against the `incoming` branch. Resolve any conflicts that arise.
2. Run the full Rust test suite with the `make check` command. You're not off the hook even if you just stick to documentation; code examples in the docs are tested as well!

Pull requests will be treated as "review requests", and we will give feedback we expect to see corrected on [style](https://github.com/mozilla/rust/wiki/Note-style-guide) and substance before pulling. Changes contributed via pull request should focus on a single issue at a time, like any other. We will not look kindly on pull-requests that try to "sneak" unrelated changes in.

Normally, all pull requests must include regression tests (see [[Note-testsuite]]) that test your change. Occasionally, a change will be very difficult to test for. In those cases, please include a note in your commit message explaining why.

Regular contributors who do not have Level 1 Access within the Mozilla project may request commit access, whereupon they will be required to sign a <a href="http://www.mozilla.org/hacking/notification/">Committer's Agreement</a> with the Mozilla Foundation.

## Getting involved: how to pick your first bug

In the issue tracker, bugs can only be assigned to people who have commit access. Also, we aspire to make as many bugs as possible "owned" by assigning them to a core Rust contributor. Therefore, just because a bug is assigned doesn't mean it's being actively worked on. We (the core contributors) are all busy, and welcome help from the community. If you see a bug you'd like to work on that's assigned but appears to be dormant, communicate with the bug's owner (by IRC, email, or with an @-reply in a comment on the issue page). If you see a bug you'd like to work on that's unassigned, it's fair game: comment to say you'd like to work on it so that we know it's getting attention.

## Conduct

* We are committed to providing a friendly, safe and welcoming environment for all, regardless of gender, sexual orientation, disability, ethnicity, religion, or similar personal characteristic.
* Please be kind and courteous. There's no need to be mean or rude.
* Respect that people have differences of opinion and that every design or implementation choice carries a trade-off and numerous costs. There is seldom a right answer.
* Please keep unstructured critique to a minimum. If you have solid ideas you want to experiment with, make a fork and see how it works.
* We will exclude you from interaction if you insult, demean or harass anyone. That is not welcome behaviour. We interpret the term "harassment" as including the definition in the <a href="http://citizencodeofconduct.org/">Citizen Code of Conduct</a>; if you have any lack of clarity about what might be included in that concept, please read their definition.
* Likewise any spamming, trolling, flaming, baiting or other attention-stealing behaviour is not welcome.

## Contributed code requirements:

Pass the existing tests. If you have a good reason for breaking a test, XFAIL it. We aim for clean builds at all times.

There is a [bot](http://bot.rust-lang.org) that builds rust. The `master` branch should be kept green.

### Pay attention to portability:
* You are responsible for clean-build condition _on all platforms_ (linux, OSX, win32), including under valgrind on linux. 
* Temporary breakage is acceptable only on a per-platform basis if you're on platform A and the breakage was on platform B that you happened to not be developing on today, and only if you fix it fast.
* If you're going to commit directly to this repo, or land other people's work into it via pull-request, please make sure you have access to all 3 primary platforms and _can_ fix per-platform breakage.
* If you are doing a lot of changes likely to cause per-platform breakage (say, a lot of linkage or threading work) please use a staging branch.

### Conform to source-formatting house style:
* 78 column maximum lines
* no tabs (except Makefiles)
* stick to local naming and code-organization style

## Communication

There is an IRC channel on irc.mozilla.org, channel #rust. You're welcome to drop in and ask questions, discuss bugs and such. It is logged at <a href="http://irclog.gr/#browse/irc.mozilla.org/rust">http://irclog.gr/#browse/irc.mozilla.org/rust</a>

There is also a mailing list at <a href="https://mail.mozilla.org/listinfo/rust-dev">https://mail.mozilla.org/listinfo/rust-dev</a>.

In both contexts, please follow the conduct guidelines above. Language issues are often contentious and we'd like to keep discussion brief, civil and focused on what we're actually doing, not wandering off into too much imaginary stuff.

## Issue tracking

Add a test to the testsuite for anything you're unsure of or see breaking in passing. See [[Note testsuite]] for details.

File bugs in <a href="https://github.com/mozilla/rust/issues">the issue tracker</a> here as well as adding tests, or instead if you can't quite figure out how to test the thing you want to point out.

Tag bugs liberally. The bug tracker here currently has weak search capabilities. Github staff has made a variety of comments suggesting "they're working on it", but in the meantime tags are our only hope. (There's also <a href="http://githubissues.heroku.com/#mozilla/rust">GHI on Heroku</a>, but it's slow.)

Tags in the tracker are in groups:

  - `[A-foo]` tags mean that the bug is in the **area** of "foo", meaning that someone who wants to work on the foo modules in the compiler should look at it. These should be pretty specific areas, not vague. Some bugs will be tagged with multiple _areas_ because they cut across areas.
  - `[B-foo]` tags mean that the bug is **blocked** in a particular state, such as "wanting clarification" or "RFC". These are effectively workflow-oriented tags, so we can see try to attack bugs that are stuck in a particular state of their life and could possibly become unstuck. Just "awaiting someone to do the work" is not a blocked state. A bug should be in _zero or one_ `[B-foo]` states, no more than one.
  - `[E-foo]` tags indicate a guess of the **effort** required by a bug. Most bugs are "medium" and don't need such tags; but some are especially easy or hard, and this can be helpful to highlight.
  - `[I-foo]` tags area subjective judgment of **importance**. A bug should be in only one `[I-foo]` state. "Wishlist" is the least important: used for non-core features that would be nice to have, but don't need to be scheduled for any particular time.
  - `[C-foo]` tags are explanations that may be applied as an explanation when a bug is **closed** without fixing it. Application of these tags is inconsistent and occasional, mostly as a form of politeness.

Add "FIXME (issue #NN): blah blah" in the source anywhere you see room for improvement, where #NN is the issue number in the tracker here. If you fix an issue on commit, remove the associated FIXMEs (grep for other occurrences) and put the exact phrase "Closes #NN" (with that capitalization) in the commit message and github will pick it up and link to the commit, close the issue.

## Language changes

If you have some vague ideas, add an entry to [[Bikesheds]]. If you have a worked-out idea, we suggest you look at the [[Note RFC Process]] to see how we manage larger changes.

Library additions are probably the most likely to be accepted. See [[Note wanted libraries]] for possible candidates.