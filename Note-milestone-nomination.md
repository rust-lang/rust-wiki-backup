Rust [milestones] are divided into two categories: point release milestones and [maturity] milestones. Point release milestones track work being done for the quarterly releases, and maturity milestones track progress toward language stability and completeness. Bugs must meet certain criteria to be assigned to milestones, and the criteria is different for each type of milestone.

For point releases, the current criteria is that *you intend to complete the bug in time for the release*. All point release milestones should be assigned to somebody.

Maturity milestones have a more formal process to establish consensus. When you see an issue that fits the criteria for a maturity milestone:

1) Add the `I-nominated` tag
2) Add a comment to the bug saying 'nominated' and, preferably, the milestone it should be added to. If you aren't certain which milestone is appropriate then that information can be left off. It's important to add the comment so reviewers can later ask about why the issue is nominated.

Do *not* assign the issue to a milestone yourself.

Every week there is a bug triage meeting. At that meeting the attendees will review the issues tagged `I-nominated` and decide whether the nomination is accepted. If accepted, the issue will be added to the milestone; if not, then a comment will be added explaining why. In either case the `I-nominated` tag will then be removed.


[milestones]: https://github.com/mozilla/rust/issues/milestones
[maturity]: https://mail.mozilla.org/pipermail/rust-dev/2013-April/003668.html