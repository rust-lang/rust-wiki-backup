# Proposal: Lightweight RFC process

In general, we'd like to ensure consensus among the core Rust team on
all changes to the language.  So, if you'd like to make a change, the
single basic requirement is to document your proposal and discuss it
as widely as you can.  The rest of this document describes the
"official" procedure for doing this.

The first step is to create an RFC issue: open an issue on github with
the "RFC" in the title and tag it with the `rfc` tag.  Others should
see the RFC tag, read the issue, and make comments.

Don't push your code to master until you've discussed the issue.
Ideally this should be at a weekly meeting in which most people are in
attendance.  If you don't want to wait that long, or you are not
getting feedback, then just send a mail to rust-dev saying, "I opened
issue XXX.  There seemed to be no disagreement.  I have now
implemented it.  I'd like to push it, is everyone ok with that?"
Assuming no one objects, you're all set.  If there are objections,
then try to address them.  Ideally we should be able to find solution
that everyone can accept.  If, in the end, consensus cannot be
achieved, we'll resolve it on a case-by-case basis.

For "user-facing" changes to the language, a detailed proposal is a
good idea.  This can be written in tandem with implementing the code:
often the best way to discover tricky cases is to implement.  You will
probably also have to make several iterations in response to feedback
or implementation challenges.  Once the group comes to general
agreement, you can start to push the code itself to master.

