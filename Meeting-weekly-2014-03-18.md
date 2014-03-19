# Agenda 3/18/2014
* rust-lang/rfcs (pnkfelix): is github inappropriate for this purpose?
* 0.10 prep

# Attending
acrichto, brson, larsberg, jack, felix, niko, cmr

# Status

- brson - fixing bots, moving bors, installation, docs, atomics
- pnkfelix - middle::cfg, rfcs
- acrichto - sync chan, landing things, reviews, libsync
- nmatsakis - reviews, RFCs

# Friend of the tree

- brson: This week's friend of the tree is Clark Gaebel. He just landed a huge first contribution to Rust. He dove in and made our hashmaps significantly faster by implementing Robin Hood hashing. He is an excellent friend of the tree.

# RFCs and GitHub

- pnkfelix: Had a conversation on IRC. The question is: is github merely weak or totally unusable? May be too soon to make a call, but wanted to bring it up. The main points that people had were: in particular, if you're not careful, you can lose comment history. So, if you force-push after a comment was on a commit, then the comment is lost, so you lose some history in the development of the RFC. The other part is that if you don't ever force-push and have a long history, if someone comments on diffs, the comments are considered outdated on github and not shown. Most importantly, you can't search through all of those comments. I was most concerned about retaining the history of the comments on the RFC, as well as the user interface. Others were concerned about migrating to a proprietary service. Mainly discussed with cmr and ChrisMorgan.
- brson: Any other venues?
- pnkfelix: cmr was looking at discourse? They found it didn't have good e-mail notification. They also looked at phabricator (from Facebook).
- larsberg: Have you looked at Critic? We use it on Servo and it integrates with Github.
- nmatsakis: Seems early to be coming up with a new RFC process, especially given the lack of a compelling alternative.
- acrichto: I've used phabricator at dropbox and facebook and it's great, though I've only used it for code reviews. Seems weird to have RFCs on one tool and other stuff in another tool.
- nmatsakis: Question is: how much do we care about the complete comment history and searchability? 
- brson: It's a bad problem for our source, too. It's just a problem with git.
- acrichto: For the RFCs, we should request commenting on diffs by going to the files changed and comment there instead of commenting through the actual commits. The bad part is that it sends far more e-mails commenting that way.
- nmatsakis: Probably ways to filter those, too.
- pnkfelix: I agree that it's early and I don't have a better solution than, say, bugzilla. So this is fine.
- brson: Agreed, let's just be aware of it.

# 0.10 prep

- brson: We have less than 3 weeks to get that released. The one thing I wanted to get in was the new installer, but I think that we won't be able to do that before we release 0.10. Risky to turn that on right before we release. I don't think there's anything else we have.

# Documentation

- brson: There was a doc sprint on Sunday. 10 people showed up in-person, pizza for about 30, and there were a bunch of people online. THe Swedish Rust meetup group even had 3 people on video conference, which was awesome! We produced about a dozen pull requests - I think we had a great result.
- azita: How often are you thinking of doing these?
- brson: Not sure. erickt is thinking maybe every other month? We should keep organizing them.

# This meeting and RFCs

- brson: We were mentioning that soon we will start talking about RFCs. Do we have any ready?
- acrichto: There are four different virtual struct ones... but maybe the attribute one we could go over.
- brson: Was that big enough to require an RFC?
- nmatsakis: Major internals refactoring.
- acrichto: There's a lint that warns you if you have unused attributes, but that doesn't work with macros. Adds ids and flags so that the linter can report only on unused attributes.
- nmatsakis: Feels a bit too imperative, as opposed to having modules declare the kinds of attributes they intend to consume... but I'm not sure.
- brson: Also assumes that attributes will not be consumed by the compiler pipeline (or cargo).
- nmatsakis: Part of the problem is not with the RFC; it's that our attributes are so undeclared. If you had to declare you attribute ala Java and where it has to be used, this would be easier. But, that would be a bunch of extra stuff we have to do. I'm not going to reject this RFC, as long as it's a side table.
- acrichto: Could switch the lint to allow by default instead of warn...
- nmatsakis: It's a legitimate problem. So long as this is feature-gated for 1.0, I definitely have no objection. I'm just a little concerned about having a binary interface to our plugins here. I'm not crazy about this as the long-term interface, but it's better than what we have.
- brson: I don't like that the whole compiler is the plugin interface.
- nmatsakis: I wouldn't want to standardize an interface that just happens to be what we have today, but that's not what we're doing, so I'm OK.
- brson: Let's do it.
- nmatsakis: I think you could declare your attribute in the future in a backwards-compatible way and still work well with warnings.
- brson: Acrichto, will you merge this?
- acrichto: Will do. 
- acrichto: The UFCS RFC hasn't gotten many comments and people seem to agree with it.
- nmatsakis: Maybe we're ready to pull that one in, then.
- acrichto: Maybe hold off one more week?
- pnkfelix: Question: just skimmed over the attribute use lint RFC, which says it doesn't distinguish between unused & unknown attributes. Is that true? Can't you run over the list... well, I understand how it handles the unused problem, but don't get why it doesn't handle the unknown problem.
- nmatsakis: What does unkown mean?
- pnkfelix: Typo. Right now, the compiler flags it.
- nmatsakis: That's the same problem. If it's not known, it's not going to get used, right?
- pnkfelix: The ones that are known and unused just weren't put in the right spot? If you had a notion of the right kinds of spots to use this attribute...
- nmatsakis: That's what I'm getting at. At least with this RFC, if you put it in the wrong spot, it won't get used and it'll get caught by the lint.
- acrichto: I think the unused vs. unknown is for error messages. If you have a feature attribute, you can't say it's unused. All you can say is that it wasn't used during compilation.
- nmatsakis: Seems like an inherent problem. Can't ever know the complete set of attributes - might have forgotten to load a plugin.
- pnkfelix: I'll think about it more and comment on the RFC.
- nmatsakis: Going to be annoying to maintain this table instead of commenting the attributes... but this table is much more flexible.
