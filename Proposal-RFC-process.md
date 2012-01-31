# Proposal: RFC process

This is intended to reduce fear, doubt, and uncertainty regarding new feature implementations. Both for the people pressing for them (discussions on the mailing list tend to peter out without clear conclusions) and for the people on whom they are unexpectedly sprung.

This process applies if you want to make a change that adds a new feature to the language, or changes the behaviour of an existing feature. For wide, sweeping changes or extremely experimental things, discussing on the mailing list still seems preferable to getting a long, unfocused discussion in the bug tracker.

This is just a proposal, feel free to comment or amend.

* Open a bug on the issue tracker, start the title with "RFC:" and tag it with the `rfc` tag.

* When github notifies you that such a bug has been opened, and you have an opinion on the subject, read it (*carefully*) and comment. If you like the proposed solution and you're a Rust developer, comment to express your support. If you don't like it, comment to say what problems you see and, if at all possible, provide constructive suggestions for adjustments to the proposal.

* RFC proposals should sit for a minimum of a week before further action is taken. Use good judgement here — things that are controversial or have an active discussion going on should be left to sit longer.

* After this period, if the comments look like a consensus (no strong objections left), go ahead an implement it.

* If not, you have three options. If a more promising modified variant of the feature/change came up in the discussion, close the old one and open a new RFC for the new one. If it looks like it just won't fly, abandon it. If you want to take another shot at convincing the team, bring it up in the next team meeting (ensuring the people with a strong opinion on the matter are present), and discuss in person.
