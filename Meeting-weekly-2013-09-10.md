# Agenda

* Policy on github commit access (tjc)
* rustpkg: `extern mod` duplication (see https://github.com/mozilla/rust/issues/8673 ) (tjc)
* rustpkg: where to store cached source code checked out from git? (tjc)
* rustpkg: what can appear in RUST_PATH? (tjc)
* * Should https://github.com/mozilla/rust/pull/8831 become the default behavior?
* * https://github.com/mozilla/rust/issues/8520 is related
* RFC on changes to implicit copyability https://github.com/mozilla/rust/issues/9098 (jack)
* 0.8
* patterns in trait and default methods (pcwalton)
* putting macro_rules behind a flag (pcwalton)
* Local::<for Trait>::take() (pcwalton)
* Sized trait vs. "unsized" prefix (pcwalton)
* drop trait destructuring idea (pcwalton)
* fate of const (nmatsakis)
* default field values for structs (pnkfelix) https://github.com/mozilla/rust/issues/6973#issuecomment-24137545

# Attending

brson, kmc, pcwalton, jack, tjc, jdm, lbergstrom, azita, nmatsakis, dherman, ecr, felix

# github commit policy

- tjc: my understanding is that graydon just added committers after pinging the whole team. now that he's not here, should we have a more formal policy for giving people commit access and whether we can give people more fine grained controls (like pushing to try).
- brson: the wiki has two sentences about this. if you want access discuss it with core contributors.
- brson: do you have thoughts on this?
- tjc: is it possible to give people more fine grained access.
- brson: we can't disentangle push to try and push to any branch right now.
- dherman: here's some github info on permissions
https://help.github.com/articles/what-are-the-different-access-permissions
- tjc: we could have a separate repo for testing.
- brson: bors needs a PR. it can't monitor arbitrary repos.
- tjc: i guess that means adding a third level of access to support try. this came up because someone wanted to push to try.
- brson: is it ok to require a PR to push to try?
- tjc: it's a little annoying but it would work.
- brson: if we did this it would revoke every everyone's push access effectively.
- niko: closing issues also require special permissions. we'd also want people to close issues.
- tjc: we could keep push access as is, modify bors to add try build support and see how that works out.
- brson: should anyone be able to do a try build? so far we haven't had a lot of trouble with abuse.
- tjc: ti would be pretty obvious if it occurred.
- brson: we can allow bors to allow trys from anyone.
- pcwalton: are the machines sandboxed? you could run any code
- brson: we'll have to add a whitelist for this. another issue is that there are so many non-mozilla peers. there's a lot of stuff going on under the radar, which is a healthy sign, but it is concerning somewhat.
- tjc: do you have specific concerns?
- brson: there are a few pull requests I wish would have been more obvious, but nothing specific.
- niko: it's hard to pay attention to all the pull requests.
- tjc: we could change the policy about what kinds of review is needed. ie, requiring core person to review.
- niko: sometimes I think we're too quick to review. I often do an initial review, but later reviews aren't as thorough. it would be good to outline as policy what areas of code need more review.
- pcwalton: there's a superreview policy in gecko. if you add public apis it requires superreview.
- niko: they also have a strong notion of ownership. you can't just ask anyone to review.
- jdm: we could set up a modules list like gecko.
- pcwalton: it doesn't need to be that formal. there's a notion that lacking an r+ is a bug, but it's not necessarily try. the reviews exist for a reason. there are some parts where we are having too slow reviews is a problem. certainly i've been guilty of r+ing things that look good and not doing thorough review.
- niko: we put in an optimization for llvm pointer aliasing that turned out to be unsound for example.
- kmc: do you think that would have been caught with our review process.
- niko: yes, i would have caught that and we did catch it later. (ed by nmatsakis: Actually, I might have reviewed that and let it through at the time...can't remember)
- pcwalton: there was the thing where people make vec::each take &const self until someone put in a commit that said "don't change this".
- niko: i fixed that at least twice. i don't think it's a huge epidemic.
- pcwalton: there is a crypto thing in the queue that i suggested we get crypto review on.
- niko: there was another crypto review i sent to graydon and he found a problem.
(discussion on getting security review from people at mozilla who are experts on security)
- tjc: do we need guidelines on how much effort you should put into something.
pcwalton: it's hard to say how much effort is needed.
- tjc: sometimes i'll think something is really straightforward and i'll say if it passes tests it shoudl be fine. thought it's possible that something will look like that and have something subtlely wrong.
- felix: my opinion has been that reviews are more for stylistic issues. i don't know if we can trust all contributors to spend that much effort.
- pcwalton: there's only so much that a human can do.
- dherman: this is defense in depth. any piece is fallible but when combined you should see fewer bugs getting through.
- tjc: just as a wild idea, maybe we could have part of this meeting where we invite contributors who have r+ to review things that have been reviewed. we should try to make them feel more a part of the core group.
- dherman: maybe a community call once a month would be a good idea.
- niko: no one's saying there's a huge amount of bad code. maybe we just need to identify some important areas that are sensitive and require a bit more review there.
pcwalton: i'm wary of making changes if we haven't noticed problems.
- tjc: i think the conclusion is that we identify areas that need more review and to change bors to a larger whitelist for people for try. are you going to do that brson.
brson: yes, i'll do that.

# extern mod duplication

- tjc; jack and i noticed this a while ago. i added `extern mod foo = 'http://....`. jack pointed out that his may lead to duplicated code. you may have to write it out in different files and they have version tags and there's no equivalent of re-exports for extern mod. is this a problem and is this clear?

extern mod foo = "github.com/mozilla/some-long-package-name#long-complicated-tag";

- jack: if we update a module right now we change one submodule pointer, but with rustpkg we might have to update N pointers to the package.
- brson: this tag specifies a sha?
- tjc: could be a tag or a sha
- jack: part of the reason to bring this up is that we don't have any bright ideas.
- jack: maybe we should have to do this and see how painful it is.
- brson: it's the same as the dozen or so version numbers scattered around.

# Implicit Copyability

- jack: there was a bug in Servo due to calling unwrap() instead of mut_ref()
- jack: this would have been fixed if the struct were not implicitly copyable
- pcwalton: what about c-style enums? they should be implicitly copyable.
- dherman: does this have to do with inherited mutability?
- pcwalton: I'm not sure that it does. When I described how implicitly copyable works to roc he was horrified.
- jack: Part of the problem is that we've standardized on unwrap(), but unwrap() doesn't always *unwrap*
...
- pcwalton: what about Option<int>?
- dherman: I think what Patrick is getting at is that there is an intution that "atomic data" should be copyable...
- nmatsakis: struct Point?
- dherman: ... "small"
- pnkfelix: What is your intution about what unwrap means? I guess I don't have one?
- jack: when you unwrap, it moves the value out, then you can't use the option anymore
- pcwalton: I was never happy with the name unwrap
- jack: it might be that unwrap is a bad name, but what do you call it, sometimes move?
- nmatsakis: we did try to have explicit move keywords everywhere, but it was quite tedious and somewhat stressful to be sure that you have move everywhere you need, but then we went the other direction
- pcwalton: I don't think that the sol'n is so easy
- brson: you could maybe define a kind bound for not 'implicitly copyable' and only define the method for that
- nmatsakis: that's a negative copy bound, maybe we could handle it
- nmatsakis: is the problem just the name?
- jack: it used to be called get, but it seems weird that it will sometimes move?
- brson: general problem with by-value
- pcwalton: we've tried a lot of directions here
- brson: not making *everything* linear
- jack: if we scope this down farther, should we make it possible to declare something as noncopyable
- brson: we have a way to do that, kind of hacky, which is this type noncopyable that has zero size
- pcwalton: very awkward for an enum
- jack: all options should be noncopyable
- kmc: I'm still confused, isn't the problem that the thing in the option is copyable?
- pcwalton: can we just make a lint mode?
- nmatsakis: I'm just not sure I want a warning at all? I feel like I want to be able to unwrap an Option<int>...
- pcwalton: there is a version of unwrap that clones the things inside. You could use that instead. so I think the lint mode might be ok.
- nmatsakis: is it only `unwrap` or are there other methods that this might apply to? everything that's by-value self?
- pcwalton: map_move, maybe? that's on iterators
- pcwalton: we could have a lint mode like "expect_move" that issues a warning
- jack: it sounds like everyone agrees that having a metadata annotation for disabling implicitly copyable is also adequate
- pcwalton: but that wouldn't fix this bug necessarily
- jack: right but I can file it and then we can debate about specific types

# Patterns, traits, and default methods

- pcwalton: Right now you can't use a pattern in default methods.
- pcwalton: Reason is this old syntax where you could drop the name of the parameter in a trait definition:

```
trait Foo {
    fn whatever(&self, int, float);
}

trait Foo {
    fn whatever(&self, _: int, _: float);
}
```

- pcwalton: supporting patterns therefore introduces an ambiguity, we work around this with a bit of lame lookahead
- tjc: should we just disallow the syntax?
- pcwalton: that's whta I propose, at worst you could write an underscore, but usually you should give a name
- brson: should we require argument names everywhere? there's at least one other place, extern declarations
- pnkfelix: extern declarations don't destructure
- kmc: definitely annoying when converting a header file into Rust
- tjc: we could require extern fns to omit the names
- pcwalton: for extern there's no real loss because extern fns can't have bodies. This really gets annoying when you have default methods.
- brson: ok, sounds like everyone pretty much agrees

# Fate of const

- nmatsakis: patrick removed const recently, which was cool.  But we still might need it
- nmatsakis: Maybe we just need a different name.  Lets see an example
[[Motivating Example]]

```
fn foo() {
    let mut flag = false;
    try(
        || ... flag = true; ...,
        || if flag { ... })
}
```

(In the above, `flag` is mutated in the try-body, and read in the handler)

- nmatsakis: it may be hard/impossible to desugar closures, as a user, into structs+methods

- nmatsakis: desugared version is logically equivalen to:

```
fn foo() {
    let mut flag = false;
    let c1 = create_closure1(&mut flag);
    let c2 = create_closure2(&const flag);
    try(c1, c2);
}
```

- nmatsakis: this would be permitted b/c mut and const can be simultaneous
- pcwalton: it does not bug me that much
- pnkfelix: I don't like the name const, I find it very unintuitive for what it means
- pnkfelix: last time that niko and I talked about this I suggested `&extern mut` but I don't really have any good ideas but from an expressiveness standpoint seems natural to want this since borrowck needs to do the reasoning for it anyhow.
- brson: is this the only case where we have this implicit const-ness
- nmatsakis: yes
- brson: can we not have it?
- nmatsakis: if you don't want flags to work, then yes
- pcwalton: my biggest worry is that people will see it and not know what they should use
- pcwalton: in most cases you *do* want to use immutable borrows, right?
- nmatsakis: yes
- brson: there was a case last week where we had multiple captures where the closure captures were &const and you didn't want it
- nmatsakis: there is still the possibility of aliasing &muts through @mut
- pcwalton: it would have fixed that particular example but I think Niko's point about flags is valid
- brson: but you're inclined not to surface it in the grammar
- pcwalton: I just don't want people to have to think about it
- brson: this will occur in closures frequently?
- pcwalton: yes but it's something the language does for you
- pcwalton: only time you'd need to do it explicitly is if you were doing something that works kinda like a closure and you are making the captures explicit, so rare as to not happen
- pcwalton: problem with &const is that it creates aliasable, mutable locations, leading to confusion, as we know
- pcwalton: I want to be sure people don't use it, but easiest way to make sure people don't use it is to not have it
- nmatsakis: I can push on the code and see how it goes, it's kind of awkward because there are types that can't be serialized and so forth
- brson: Will it show up in error messages?
- nmatsakis: I think not, but I'm not sure
- jack: if we disallowed the const borrows at all, would it invalidate code?
- nmatsakis: yes, some

# 0.8

- brson: scheduled for 3 weeks for now, not sure what that means for us, there are no big features planned, but we should do some dry runs to make sure the process goes smoothly
- brson: need a snapshot for sure, not valgrind clean, I will work on the valgrind issues
- brson: ok, thanks





