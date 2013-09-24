# Agenda 9/24/13
* continue (brson)
* comment nesting (pcwalton)
* attribute syntax (brson)
* struct-like enum variants (brson)
* overflow protection (brson)
* Lint reasons (tjc) https://github.com/mozilla/rust/pull/9025
* where can braces be dropped? (brson) https://github.com/mozilla/rust/issues/7981
  * "impl A;" vs "impl A {}", #9336 (acrichto)
* removing AUTHORS.txt (brson), #5037
* Clone -> Copy (brson)
* surprises in resolve, #8215 (acrichto)
* % vs \ as an escape in format! (acrichto)
* priv as a keyword, struct fields/enum variants (acrichto)
* drop in static items (acrichto)
* Removing linkage attributes for rustpkg crates - https://github.com/mozilla/rust/issues/8523 (tjc)
* r"raw strings" (pnkfelix): is regular language a hard requirement for lexer?
  * https://github.com/mozilla/rust/issues/9411
  * (question was implicitly answered by discussion on nested comments)
* IRC RFC (pnkfelix): e.g. #rustc and #rust-libs for (compiler/lib) internal dev vs. rust users
  * lots of traffic in #rust, especially to assist beginners
  * great to see excitement; but can impede specific technical conversation.  So #fork

## 0.8 status
- brson: ready to go and have the snapshot. regressions in the runtime and bad windows installation situation. Fixing is a lot of work and we are not going to wait for it. Objections on the Windows version?
- pnkfelix: what's wrong?
- acrichto: horrific mingw installation scenario for bootstrapping
- pcwalton: it's bad, but not a blocker
- brson: now have 3 Windows users working on it, but they all know the workarounds. Going with it.

## continue
- brson: can we change loop to continue?
- tjc: it was loop before to keep the keywords short
- brson: no objections; done

## struct-like enum variants
- brson: we don't use them; can we put them behind a flag? great.

## lint reasons issues
- tjc: patch submitted, but do we want to include this feature? It might be overkill.
- acrichto: it's not used anywhere now and is only for diagnostics, so inline comment seems to suffice.
- brson: same thing when we ignore a test. Since it doesn't care, I throw reason=something in there.
- acrichto: if we looked at the contents, this would be useful. But without anything to look at them, seems superfluous.
- pnkfelix: hard to put in place a feature without the implementation in place. 
- pcwalton: like the feature, but don't have a strong opinion
- pnkfelix: maybe have another lint that filters using this machinery, but it's missing right now
- nmatsakis: could print in the documentation, but that's not a good fit for rustdoc
- tjc: let's not do this now

## braces
- acrichto: struct and impl can have ; after them. instead, we should just require them after tuple structs but everything else will require braces. Any strong feelings? We could also consistently allow ;s. 
- tjc: the limited scenario sounds like the sensible one
- nmatsakis: I'd like not to allow both so that in the future we could use that to mean something.
- acrichto: drop "impl A ;" and require empty {}
- pcwalton: that's still allowed? my fault!

## comment nesting
- pcwalton: I want them.
- tjc: me too
- dherman: <thumbs up>
- pcwalton: there is an IDE problem, but that's just broken IDEs. I think there's one idiom that nested comments break.
- acrichto: how should it show up in documentation?
- tjc: it just shouldn't exist. for rustdoc comments inside a comment
- nmatsakis: it's just a /*, so it should just appear as normal if the comment is nested inside a rustdoc comment
- pcwalton: missing it in doc comments is really bad
- dherman: I just want to be able to comment out a region that may have a comment in it

## raw strings
- pnkfelix: since a regular language isn't an issue (see above!) no issue here

## IRC
- pnkfelix: time to split channels for #rustc/#rustlib because of the heavy noise in #rust
- acrichto: does firefox do something similar?
- nmatsakis: javascript, etc. do
- pcwalton: #rust and #rust-dev would probably be enough
- pnkfelix: #rust-dev = "rust developer" or "rust user who develops"
- dherman: already have the rust-dev mailing list
- nmatsakis: that's the problem - that list is not used
- ?: #rust-internals maybe
- pcwalton: maybe just #rustc, and talk about the libraries there too. or #rustapi.
- acrichto: I like #rust-internals
- tjc: or create #rust-beginners?
- brson: easier to move the few of us. Or rather clone() us, not move. #rust-internals sounds fine

## authors.txt
- brson: mostly complete list of everybody who's contributed to rust. But Git also has that info. Can we just remove the file?
- tjc: auto-generated file to include with the release?
- brson: no, you use a 1-line git command
- pcwalton: "mkcredits"
- tjc: we could just build it out of the history. people value having their name in the file.
- brson: we include the list of contributors in the release announcement. That won't change.
- acrichto: maybe a makefile rule to create it, and just put that in the tarball.
- pcwalton: somebody will have to make that, and I sympathize
- tjc: could be a rust program to generate it
- dherman: putting in a little effort to show appreciation to the community important
- pcwalton: ala about:credits in firefox
- tjc: darcs has an author substitution file to correct names to the canonical one
- acrichto: there's a bug for this already
- dherman: if it's a lot of work to be able to lose the file, then let's just keep the file. a week for a state-of-the-art authors.txt is not worth it
- tjc: maybe we can find something. that is not a Haskell program...

## linkage attributes
- tjc: some of the linkage attributes overlap with what can be inferred. on the other hand, having separate rules for rustpkg crates and non-rustpkg crates. Should there be a version attribute? Required? Warning if it could have been inferred from git or the directory name?
- brson: weird to have a language feature tied to git
- tjc: could be whatever VC system.
- brson: need non-VC'd code to be able to have a version. e.g., tarballs.
- tjc: a rustpkg crate does not have link attributes. a non-rustpkg crate (compiling with rustc) you can have name and version. UUID seems unneeded. Also URL. rustpkg knows what those two are
- acrichto: important for rustdoc as well. can rustdoc call on rustpkg to get this information?
- tjc: it's a library with an API for that
- brson: but rustpkg might want to use rustdoc... more crate factoring ahead!
- tjc: we can design for that. so it's OK to say that rustpkg will error out if there are linkage attributes
- jack: versions aren't easy to infer. either from the tarball name with it in the directory or from the VCS. lots of burden on small package developer to tag their releases.
- tjc: then it'll default to 0.1. Which may not be the best option.
- brson: how about allow both and rustpkg errors if they disagree?
- tjc: so you can have the attribute and the tag in git? sure, but still disallow the name attribute in rustpkg.
- pnkfelix: why not the same policy as with version?
- tjc: it's easier to infer the name. not tag.
- pnkfelix: why penalize someone for writing down the right thing?
- jack: rustpkg doesn't crawl the whole system getting the link metadata from the system. it searches by directory structure. For something named foo, it will only look in the foo directory and will never look for something named foo in the bar directory.
- tjc: that's why it should error if the attributed name is inconsistent with the inferred name. I'm fine with forbidding it, but pnkfelix might want it
- brson: afraid of code being invalid base on whether the code is in VCS or not
- tjc: only affects name, which is controlled by the parent directory or the URL for things fetched from a remote repository
- brson: my concern is the same. do the sanity check for both of them
- tjc: but still get rid of UUID/URL
- brson: nobody does anything with them, right? related, should we enforce what goes in the link attribute? they're open and included in the hash...
- tjc: more stuff like "TimsCoolAttribute=3" would be in the hash...
- acrichto: not sure on the use case, but it makes sense

## open PR to rename clone to copy
- brson: intention is good. but hugely disruptive. What are the feelings people have here?
- pcwalton: I though we should do it, but now I'm used to clone and indifferent.
- tjc: fine with
- dherman: our clone is a sheep clone, right?
- tjc: what does that mean?
- dherman: both shallow and deep. not a totally surface clone, but also not a transitive closure copy. It's a copy of just the one conceptual data structure. Deep clone is usually out of control. But shallow clone is also usually dumb. Sheep clone is hard, but we actually have it. There's a research precedent for the name clone here, but that's not sufficient to stop us.
- acrichto: are we just trying to save a letter?
- pnkfelix: the reason is to avoid confusion for people coming from Java. It's overridable, so could be deep, could be shallow, could be sheep. Built-in is a memcpy() there.
- tjc: not that worried about Java programmers being confused about that.
- pcwalton: Dave's argument is good - there is a historical name for this, so we should just 
stick with it.
- dherman: sounds like a bikeshed nobody really wants to paint. leave it alone.

## 8215 - surprises in resolve
- acrichto: all of the surprises are on what's public vs. private. If you're looking a cross-crate, it has to be public all the way down. If it's intra, the destination must be public. I'm going to rip that out to make things simpler. It's sometimes implied that you can always access your immediate parent's items regardless of public vs. private, which seems sketchy.
- nmatsakis: intra-crate, the only thing that's necessary is that the destination is public?
- acrichto: or it's in the same module
- nmatsakis: that's not what we're supposed to have now.
- acrichto: it's different than what we have
- nmatsakis: don't like the asymmetry. it should all feel like a unified module.
- acrichto: so you would like public all the way down? Or just same rule for both?
- nmatsakis: use case: crate-wide visibility. but do we need to conflate that with pub? I'd like to be able to have a public interface and then a bunch of private implementation externally that's visible everywhere internally. So the parent would be open, but you can't then go down without public.
- acrichto: but then you can't export child modules because they have to be public
- nmatsakis: up zero or more times is something you can name/access. Then, it's whatever is public within those. 
- pcwalton: as long as those helpers are public, it doesn't violate the rules. Your parent could not access those helpers, though, because a step into you and then further down would not work unless it was public.
- acrichto: so you can currently step into children even if they are private?
- nmatsakis: yes. in my view, pub means accessible by the outside world. uncles/aunts etc.
- pcwalton: it's the right thing to do, but the fallout means that globs will start importing private things.
- acrichto: I tried to do that, but it broke everything. So we might have to leave public/private inside there because it's terrible.
- pcwalton: if the whole point is to move privacy out of resolve, that's a problem.
- tjc: remove globs?
- brson: some people like them; some want to ignore them. hide it behind a feature flag
- nmatsakis: we should figure out what they're supposed to do
- acrichto: globs bring in too much. currently they bring in re-exports
- pcwalton: that's the problem: only want to bring in pub use stuff
- nmatsakis: use should be a macro, not something that's exported
- pcwalton: pulling in all items public or private is fine, but only pub uses, not private uses
- acrichto: I'll try that. Before I go further on the resolve, I'll write something up before I do that, for the next meeting.

## overflow protection
- brson: important feature for safety. We should try to turn trapping for overflows on by default and then have methods for wrapping math and see what the performance impact is.
- tjc: sounds good
- brson: big change. Lots of people will have feelings on it.
- acrichto: so, do we raise a signal? fail the task? condition? let's try it out to see what happens.
- pcwalton: have to be able to turn it off
- brson: some languages have block-level statements
- pcwalton: C#-style "unchecked{...}"
- pnkfelix: so is that in unsafe blocks?
- pcwalton: somebody discovered our vec implementations are all in a lot of trouble because we don't check and they might overflow. unsafe blocks are the most important place, since in safe code, it's just an error. in unsafe, you're using pointers and indices.
- brson: surprised there isn't much opposition
- pcwalton: nervous, mainly from code size. Should look at the research to keep the code size down.
- brson: aren't we beholded to LLVM?
- pcwalton: maybe for signed and not unsigned, but that doesn't help
- acrichto: how does this work?
- pcwalton: call an LLVM intrinsic. possibly lots more code.
- pnkfelix: maybe clamped instead of failing?
- pcwalton: probably the same in terms of the resulting change in code size
- pnkfelix: I was thinking in terms of behavior
- pcwalton: probably not any better for stopping exploits
- nmatsakis: new "unchecked" keyword?
- brson: I feel like when you're doing this it's going to be rare.
- pnkfelix: maybe checked type vs. unchecked type?
- pcwalton: could do that
- nmatsakis: always liked the idea that u32/u64 do but i32/i64 do (check, that is)
- pcwalton: then what's the default?
- pnkfelix: that gives us an incremental path
- nmatsakis: and then switch it out? I'd like to know what the perf impact is because that will affect this discussion.
- pcwalton: no path forward without an experiment
- acrichto: so we're not against it, but we need to measure it

## % vs. \ as escapers in format! strings
- tjc: percent
- acrichto: the slashes turn into something pretty hideous
- acrichto: need to escape braces... first pass through the rust string parser, so you need double-backslashes, then again in the format string escaper. % would save us, until you get to braces. But raw strings help us get around this. I still prefer slash, but it's really bad unless we get raw strings.
- brson: raw strings have momentum
- acrichto: I think going ahead with raw strings lets us ignore the changes to escapes
- pnkfelix: wanted to support user-specified delimiters with a fallback to ours. If we're willing to put in raw strings, \ is fine.
- acrichto: I'll postpone until after the raw string discussion is complete.
- brson: yeah, since \ works

## more private
- acrichto: make struct fields private by default
- dherman: when you say you can't have a public enum with private variants, you can however have a private struct with public fields. OK, so you have abstract data types that you can export that only you can create. And the enum thing makes sense because you can't pattern-match on an enum if you can't see the variants.
- acrichto: that's why it exists
- brson: internal messages in servo
- nmatsakis: seems kind of inconsistent between structs and enums. then you have pub everywhere that you want it except enum variants.
- jack: tuple structs are going to be weird
- pnkfelix: wait so even a pub struct means I have to put pub everywhere? I don't like this.
- pcwalton: people are going to write Point and be confused
- brson: we have the wrong default. looking at our standard library, we have way too much public stuff.
- acrichto: but everything else in Rust is private by default except struct fields.
- nmatsakis: and enum variatns
- acrichto: but you can return a private struct and use the private fields
- pnkfelix: if you just said struct and did a pub use to export it, then you'd have private fields. but if I say pub struct I expect people to use it.
- tjc: I would expect to have to say it
- acrichto: but with a hash map, you don't want somebody touching your buckets...
- brson: save this for next time?
