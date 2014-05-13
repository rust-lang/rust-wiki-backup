Development policy for Rust.

## Contributions and commit access

We request that contributions are made via pull request and review. Note that **all patches from all contributors get reviewed**. After a pull request is made other contributors will offer feedback, and if the patch passes review someone with official review privileges will accept it with an "r+" comment. Upon successful review, pull requests are tested and merged automatically by our integration bot, [bors](http://buildbot.rust-lang.org/bors/bors.html). When pull requests fail integration testing authors are expected to update their pull requests to address the failures until the tests pass and the pull request merges successfully.

Those with review privileges or direct push access are required to [file a Mozilla "committer agreement"](https://www.mozilla.org/hacking/notification/). This is a short (1-page) legal agreement that asserts certain claims of originality, absence of conflict with other agreements, and similar terms associated with licensing and authorization. If you already have Level 1 or higher commit rights within the Mozilla project, you have already filled out such a form. **Committer agreements are generally only required for reviewers and those with direct push access to the master repository**, though we will occasionally also request agreements for large, non-trivial contributions. If you would like to be a reviewer then please discuss in with one of the [core contributors](Note-core-team) through IRC or email.

## Getting involved: how to pick your first bug

In the issue tracker, bugs can only be assigned to people who have commit access. Also, we aspire to make as many bugs as possible "owned" by assigning them to a core Rust contributor. Therefore, just because a bug is assigned doesn't mean it's being actively worked on. We (the core contributors) are all busy, and welcome help from the community. If you see a bug you'd like to work on that's assigned but appears to be dormant, communicate with the bug's owner (by IRC, email, or with an @-reply in a comment on the issue page). If you see a bug you'd like to work on that's unassigned, it's fair game: comment to say you'd like to work on it so that we know it's getting attention.

## Pull request procedure

To make a pull request, you will need a Github account; if you're unclear on this process, see Github's documentation on [forking](https://help.github.com/articles/fork-a-repo) and [pull requests](https://help.github.com/articles/using-pull-requests). Pull requests should be targeted at Rust's `master` branch. Before pushing to your Github repo and issuing the pull request, please do two things:

1. [Rebase](http://git-scm.com/book/en/Git-Branching-Rebasing) your local changes against the `master` branch. Resolve any conflicts that arise.
2. Run the full Rust test suite with the `make check` command. You're not off the hook even if you just stick to documentation; code examples in the docs are tested as well!

Pull requests will be treated as "review requests", and we will give feedback we expect to see corrected on [style](https://github.com/mozilla/rust/wiki/Note-style-guide) and substance before pulling. Changes contributed via pull request should focus on a single issue at a time, like any other. We will not look kindly on pull-requests that try to "sneak" unrelated changes in.

Normally, all pull requests must include regression tests (see [[Note-testsuite]]) that test your change. Occasionally, a change will be very difficult to test for. In those cases, please include a note in your commit message explaining why.

The copyright header at the top of the file should include a date range that includes any years in which the file was changed in a significant way. For example, if it's 2013, and you change a file that has existed since 2010, it should begin

```
// Copyright 2010-2013 The Rust Project Developers.
```

## Conduct
* We are committed to providing a friendly, safe and welcoming environment for all, regardless of gender, sexual orientation, disability, ethnicity, religion, or similar personal characteristic.
* On IRC, please avoid using overtly sexual nicknames or other nicknames that might detract from a friendly, safe and welcoming environment for all.
* Please be kind and courteous. There's no need to be mean or rude.
* Respect that people have differences of opinion and that every design or implementation choice carries a trade-off and numerous costs. There is seldom a right answer.
* Please keep unstructured critique to a minimum. If you have solid ideas you want to experiment with, make a fork and see how it works.
* We will exclude you from interaction if you insult, demean or harass anyone. That is not welcome behaviour. We interpret the term "harassment" as including the definition in the <a href="http://citizencodeofconduct.org/">Citizen Code of Conduct</a>; if you have any lack of clarity about what might be included in that concept, please read their definition. In particular, we don't tolerate behavior that excludes people in socially marginalized groups.
* Private harassment is also unacceptable. No matter who you are, if you feel you have been or are being harassed or made uncomfortable by a community member, please contact one of the channel ops or any of the [Rust core team](Note-core-team) immediately. Whether you're a regular  contributor or a newcomer, we care about making this community a safe place for you and we've got your back.
* Likewise any spamming, trolling, flaming, baiting or other attention-stealing behaviour is not welcome. 

## Moderation

These are the policies for upholding our community's standards of conduct in our communication channels, most notably in Rust-related IRC channels.

1. Remarks that violate the Rust standards of conduct, including hateful, hurtful, oppressive, or exclusionary remarks, are not allowed. (Cursing is allowed, but never targeting another user, and never in a hateful manner.)
2. Remarks that moderators find inappropriate, whether listed in the code of conduct or not, are also not allowed.
3. Moderators will first respond to such remarks with a warning.
4. If the warning is unheeded, the user will be "kicked," i.e., kicked out of the communication channel to cool off.
5. If the user comes back and continues to make trouble, they will be banned, i.e., indefinitely excluded.
6. Moderators may choose at their discretion to un-ban the user if it was a first offense and they offer the offended party a genuine apology.
7. If a moderator bans someone and you think it was unjustified, please take it up with that moderator, or with a different moderator, **in private**. Complaints about bans in-channel are not allowed.
8. Moderators are held to a higher standard than other community members. If a moderator creates an inappropriate situation, they should expect less leeway than others.

In the Rust community we strive to go the extra step to look out for each other. Don't just aim to be technically unimpeachable, try to be your best self. In particular, avoid flirting with offensive or sensitive issues, particularly if they're off-topic; this all too often leads to unnecessary fights, hurt feelings, and damaged trust; worse, it can drive people away from the community entirely.

And if someone takes issue you with something you said or did, resist the urge to be defensive. Just stop doing what it was they complained about and apologize. Even if you feel you were misinterpreted or unfairly accused, chances are good there was something you could've communicated better — remember that it's your responsibility to make your fellow Rustians comfortable. Everyone wants to get along and we are all here first and foremost because we want to talk about cool technology. You will find that people will be eager to assume good intent and forgive as long as you earn their trust.

*Adapted from the [Node.js Policy on Trolling](http://blog.izs.me/post/30036893703/policy-on-trolling)*

## Contributed code requirements

Pass the existing tests. If you have a good reason for breaking a test, XFAIL it. We aim for clean builds at all times.

There is a [bot](http://buildbot.rust-lang.org/) that builds rust. The `master` branch should be kept green. The [bors](http://buildbot.rust-lang.org/bors/bors.html) bot merges pull requests after they have been signed off (`r+`'ed) by an authorized reviewer.

### Pay attention to portability:
* You are responsible for clean-build condition _on all platforms_ (linux, OSX, win32), including under valgrind on linux. 
* Temporary breakage is acceptable only on a per-platform basis if you're on platform A and the breakage was on platform B that you happened to not be developing on today, and only if you fix it fast.
* If you're going to commit directly to this repo, or land other people's work into it via pull-request, please make sure you have access to all 3 primary platforms and _can_ fix per-platform breakage.
* If you are doing a lot of changes likely to cause per-platform breakage (say, a lot of linkage or threading work) please use a staging branch.

### Conform to source-formatting house style:
* 100 column maximum lines
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

Tags in the tracker have [specific definitions](Note-tag-label-names-and-definitions) and in groups:

  - `[A-foo]` tags mean that the bug is in the **area** of "foo", meaning that someone who wants to work on the foo modules in the compiler should look at it. These should be pretty specific areas, not vague. Some bugs will be tagged with multiple _areas_ because they cut across areas.
  - `[B-foo]` tags mean that the bug is **blocked** in a particular state, such as "wanting clarification" or "RFC". These are effectively workflow-oriented tags, so we can try to attack bugs that are stuck in a particular state of their life and could possibly become unstuck. Just "awaiting someone to do the work" is not a blocked state. A bug should be in _zero or one_ `[B-foo]` states, no more than one.
  - `[E-foo]` tags indicate a guess of the **effort** required by a bug. Most bugs are "medium" and don't need such tags; but some are especially easy or hard, and this can be helpful to highlight.
  - `[I-foo]` tags area subjective judgment of **importance**. A bug should be in only one `[I-foo]` state. "Wishlist" is the least important: used for non-core features that would be nice to have, but don't need to be scheduled for any particular time.
  - `[P-foo]` **priority** tags are used for release planning. These are the only tags that should not be applied based on individual judgement - they are instead applied as part of the triage process, described below.

Add "FIXME (issue #NN): blah blah" in the source anywhere you see room for improvement, where #NN is the issue number in the tracker here. If you fix an issue on commit, remove the associated FIXMEs (grep for other occurrences) and put the exact phrase "Closes #NN" (with that capitalization) in the commit message and github will pick it up and link to the commit, close the issue.

## Milestone and priority nomination and triage

While Rust point releases are time-based (not feature-based) we still prioritize which issues to work on, particularly as we progress toward the first major release. High-priority issues are tagged with the `P` tags, and optionally assigned to a major release milestone (which are feature-based). Issues that impact backwards compatibility are always high priority, but there are [additional criteria](Note-priority-issue-criteria) as well.

When you see an issue that fits the criteria:

- Add the `I-nominated` tag
- Add a comment to the bug saying 'nominated' and, preferably, the milestone it should be added to. If you aren't certain which milestone is appropriate then that information can be left off. It's important to add the comment so reviewers can later ask about why the issue is nominated.

Do *not* assign the issue to a milestone or a priority tag yourself.

Every week there is a bug triage meeting. At that meeting the attendees will review the issues tagged `I-nominated` and decide whether the nomination is accepted. If accepted, the issue will be added to a milestone and/or tagged; if not, then a comment will be added explaining why. In either case the `I-nominated` tag will then be removed.

## Language changes

Major changes (including all language enhancements) must go through the [RFC process](https://github.com/rust-lang/rfcs/blob/master/active/0001-rfc-process.md).
