# Agenda 4/15/2014

* The priv keyword (acrichto) https://github.com/rust-lang/rfcs/pull/26
* tweaked variance (acrichto) https://github.com/rust-lang/rfcs/pull/12
* Smaller refcounts (acrichto) https://github.com/rust-lang/rfcs/pull/23
* attributes on statements/match arms (acrichto) https://github.com/rust-lang/rfcs/pull/16
* completing an rfc (acrichto) https://github.com/rust-lang/rfcs/pull/30
* module system rfc (nmatsakis)  https://github.com/rust-lang/rfcs/pull/18
* assigning PRs (nmatsakis)
* Favoring Vec<T> or ~[T] (pnkfelix) https://botbot.me/mozilla/rust-internals/msg/12851440/ 
    see also:   http://www.reddit.com/r/rust/comments/21y9tw/meetingweekly20140401_vect_vs_t_virtual_structs/cghztkv  
* Design FAQ (cmr) https://etherpad.mozilla.org/VDM304DpBn
* Deref/DerefMut conventions (acrichto) https://github.com/rust-lang/rfcs/pull/25
* unsized/type/Sized? keyword (nrc)
* bounds on type vars in structs and enums (nrc) https://github.com/rust-lang/rfcs/pull/20
* Arc<Mutex<T>> bounds (brson) https://github.com/mozilla/rust/issues/13125
* ~"foo" (pcwalton)

# Status

acrichto - upgrading buildbots, freebsd madness, lots and lots of bugs

# Action Items

- brson - Accept RFC 26
- acrichto - email about breaking changes in commit logs
- acrichto - follow up on attributes on macro arms/statements RFC
- cmr - PR for the design FAQ etherpad

# ~"foo"

- pcwalton: I want to remove the owned string literal notation. The reason is that it's the one thing that isn't compositional with DST. Because "" without the sigil gives you &str. Otherwise, you would get ~&str.
- nmatsakis: Alternatively, the behavior of this will change with DST.
- pcwalton: I was assuming it wouldn't and we'd have to hack it in the parser to still be ~str.
- nmatsakis: Specifically, I was hoping to handle this without special-casing it in the parser.
- pcwalton: So, no way to create an ~str without .to_str().
- nmatsakis: You could do:
```
box "foo" -> Heap<&'static Str>
"foo".to_owned() -> Heap<Str> (or ~Str)
Heap::from("foo") -> Heap<Str> (or ~Str)
Rc::from("foo") -> Rc<Str>
```
- acrichto: You can't say box("foo"), because you'll get a ~&str, right?
- pcwalton: There will be no built-in syntax for creating an owned string, which is good because pople create them too much.
- acrichto: So, string.to_owned.
- nmatsakis: I just use format!
- pcwalton: It's an anti-pattern to have the literals be too easy to make because they're more expensive than they look.
- acrichto: We can remove a lang item.
- nmatsakis: And it makes trans cleaner, which is always a good sign.
- pcwalton: uniq_str_eq is another bad lang item. I think it was for pattern matching on unique strings. We would replace this with .to_owned().
- brson: Seems uncontroversial.
- acrichto: It'll be painful...
- brson: Can you mail to the mailing list, pcwalton? 
- nmatsakis: Seems like a good candidate for the breaking changes log. Having that log in the repo would help - what changed, and how do you change it.
- pcwalton: Yes, it'll affect all Rust code.

# Breaking changes log

- brson: So, are we officially making a decision about this? 
- nmatskais: We should add a file to the repo. Any time you make a breaking change that will break downstream users, you should add a note into the file about 1) what has changed and 2) how to update your code. I know they're partially in TWiR, but they're not comprehensive.
- brson: This is libraries too, right?
- nmatsakis: Yes, unless it's a super-obscure method.
- acrichto: Lots of conflicts to this file. Every PR will conflict on it. In theory the change log is the commit log...
- nmatsakis: It's a subset. Maybe part of the commit message and we extract it.
- acrichto: Yes, maybe put it in the PR with a special format.
- nmatsakis: And then it's just a perl script.
- brson: Have to get the reviewers on board to have this in there.
- nmatsakis: We should send some mail.
- brson: Maybe the script is not sufficient? Still needs work to procure the info?
- nmatsakis: What would be better? 
- brson: If we automatically ran the script and posted it, that would be fine.
- acrichto: I'd expect that the work <breaking> would have a nice bit about the update. So then you can just get the git log and grep for messages with it.
- nmatsakis: All caps?
- brson: NO!
- acrichto: #breaking, maybe...
- brson: Who will make this happen?
- acrichto: I can write up an e-mail.
- nmatsakis: Or I will do so.

# RFC #26, removing priv

- acrichto: We have removed private fields, one of the remaining left. Private enum variants are the only thing left. This RFC removes them, with no replacement.
- brson: I'm in favor, as long as we reserve the keyword. How much code does this break?
- acrichto: Some debuginfo code.
- brson: Probably some in Servo, too. But is anybody against this RFC? No? Let's do it. Who volunteers to accept it?
- cmr: Who has push access to the RFC repo?
- brson: Me, alex, pcwalton, and niko. I'll accept it.

# Mutex bounds

- brson: We fixed a bunch of problems with Mutex, but there is an issue on the bounds of the Mutex - they're just not right anymore.
- acrichto: Mutex<T> should be Share even if T is not Share. So we need Unsafe<T> to always be Share. I think we have a way forward, but just haven't implemented it.
- nmatsakis: It's always Share if the content is Send, right?
- acrichto: Yes. Also Mutex requires a Send bound, but it should be Send or Share...
- nmatsakis: I thought you'd just use RWArc if you want Share. We could potentially address it by allowing multiple impls for that.
- acrichto: The most relevant issue is that Mutex<T> is not Share if T is not Share, but that will get fixed with the Unsafe special treatment.
- brson: How does this work?
- acrichto: Everything about Mutex that it contains is Share except for the T. And the T is in an Unsafe<T>. That's inferred to be not- Share if T is not Share. The change would be that if T is Send then Unsafe<T> is Share.
- nmatsakis: With opt-in, built-in traits, this would be more explicit... Mutex would say:
```
impl<T:Send> Share for Mutex<T> { ... }
```
- brson: With opt-in kinds, Mutex would just say that's it's Share if T:Send, right?
- nmatsakis: Unsafe<T> does not present any abstraction you can use because it's an unsafe building block... I think there's no logic required. Maybe could do it on Unsafe<T> instead:
```
Unsafe<T> could be one of the following
// What Alex proposed would really be explained by two distinct impls:
impl<T:Send> Share for Unsafe<T> { ... }
impl<T:Share> Share for Unsafe<T> { ... }
// If we don't want to support multiple instances in this case, we could do this:
impl<T> Share for Unsafe<T> { ... }
```
- nmatsakis: don't know if we need it, though. 
- acrichto: I think Unsafe will have to be special rules backed into the compiler, though.
- nmatsakis: Neither is particularly safe in that Unsafe<T> does not enforce threading restrictions. But given something that's neither Send nor Share, I don't know how to make it work. Maybe we should just add those Share implementations on Unsafe... I see the question of whether to permit both as kind of separate from how to make Mutex functional.
- brson: Looks like we have a solution, at least that should work until we have opt-in kinds. Do we need an RFC? Or is this just a bugfix?
- acrichto: Just a bugfix.
- brson: There's an issue. Need it fixed soon because it's blocking servo.
- nmatsakis: Not too hard to fix, I hope.
- brson: I feel good about this!

# RFC 16 (attributes on macros/match arms)

- acrichto: Add attributes to statements and match arms. Original was off of enum variants, and then the question was: why not more things? The open question was the else block of an if-statement.
- brson: So, this RFC is all up to date with the comments?
- acrichto: Yes.
- brson: So this adds attributs to a lot of things, right?
- acrichto: Match arms and statements.
- cmr: if/else is one of the unsolved questions.
- brson: So this is not quite ready to merge?
- cmr: I'd be willing to take it if we punt on what to do around if.
- brson: Issue with if is simply because of else?
- nmatsakis: What's the problem?
- brson: Can't put attributes on else branches.
- nmatsakis: because of elseif?
- acrichto: First is: where do you put it. Second is if...else is not always a statement. The main reason for this was that match arms was off of enum variants. Now, you can just say #[cfg(unix)] on statements.
- nmatsakis: Was the goal only to attach to statements?
- acrichto: Yes. Statements and match arms.
- nmatsakis: Well, all of these play multiple roles... match arms can appear outside a statement. Are they forbidden in that catch?
- acrichto: They are special. 
- nmatsakis: I don't understand why we would limit it to statements. It seems kind of arbitrary, e.g. "if at outermost level." I can see not wanting to put in on random expressions like `1+2`... wait, no, now I don't see.
- brson: Can't we just parse attributes every time we parse expressions?
- nmatsakis: Maybe some ambiguities here?
- brson: Basically a special form of attributes on expressions, since statements are for the most part expressions.
- nmatsakis: There will be a schism in the grammar. But maybe that's not entirely true? In the AST, there will be a place for them on all ifs, otherwise there will be two If AST nodes.
- acrichto: Can configure a lint for one statement instead of a whole function. The main reason on ifs was for branch prediction... maybe some hinting?
- nmatsakis: I don't know what people will use them for; that's part of the design. Though it would be nice if they were namespaced.
- brson: Are you against this?
- nmatsakis: No, I don't object to what's written here. I just want to make sure we're uniform.
- brson: These trialing expressions will be attribute syntax expressions, not attribute syntax statements. Tail expressions will be expressions, not statements.
- nmatsakis: Seems OK. Have the tooling even if we don't let you type it... so I don't know why we don't let you type it.
- brson: This proposal requires the AST to have attributes on every expression and statement.
- nmatsakis: Not all of them? Only need it if the expression is an if/match/call, right?
- brson: Could... 
- cmr: I don't see why the attribute would be attached to the expression, when we have a statement.
- nmatsakis: Things people think are statements are not AST statements:
```
fn foo() {
    if cond { println!("Hi"); } // <-- `if` is not a statement here but rather a tail expression
}
```
- cmr: I think the restriciton was for simplicity.
- nmatsakis: I like the Rust design beign that every statement is an expression. I'd like to be able to write this:
```
let bar = if VERY_LIKELY { 0 } else #[cold] { 1 };
```
- brson: Even if you could tag all expressions, you couldn't tag that... this is a tangent.
- nmatsakis: No! My point is that if I have some annotation that I want to attach to an if unless I rewrite it, as below:
```
let bar = #[cold] if VERY_LIKELY { 0 } else { 1 };
becomes
let bar;
#[cold]
if VERY_LIKELY { bar = 0; } else { bar = 1; }
```
- acrichto: One option is not just statements. Maybe just match arms, for now?
- nmatsakis: Why not all if/match/call expressions? Are people uncomfortable with that?
- brson: BIG parser change. 
- nmatsakis: Either is a big change to the parser. I think I'm both afraid of the minimal change and of having an incomplete change.
- brson: Then I think maybe pareing this back per acrichto to just match arms will make this better.
- acrichto: For me, the strong rationale is that we allow them on enum variants, so we should probably do it on match arms as well because it's where we look at them.
- nmatsakis: But match arms are on declarations as well. True/false, 1/2, etc. I don't object to the smaller thing because we're afraid of the big thing... but it'd be nice to have an investigation of if the bigger thing causes problems.
- acrichto: If we start allowing it in a few places, we will get pressure to put it everywhere.
- nmatsakis: I could imagine it being useful. Specifically, Java lets you limit your linting to a block. We can limit to a function now, but it'd be nice to be able to do that on a single assignment, etc.
- brson: Shoudl we try to make that happen?
- nmatsakis: No. I'd like to go with the match arms for now. But if we do more than that, I'd like to see why we can/can't do it.
- brson: Maybe follow up with huonw?
- nmatsakis: Sooner or later, I'd like to investigate giving lifetime names to all expressions. That is similar, so maybe I would find out what happens then.

# Deref/DerefMut PR

- nmatsakis: I think it's too early to have an official convention here. I'll write that on the PR.

# RFC 30 completing an rfc 

- acrichto: How do we complete them? Or just merge them?  Because I just move them from active to complete.
- brson: You can just do that.
- nmatsakis: I agree.

# Design FAQ 

- cmr: Some things come up over and over in IRC. This is more about specific things we don't want to change. It may have overreached a bit - no exceptions, etc. But I wanted to bring this up - is it something that belongs in the wiki, repo, etc? And is it wrong?
- acrichto: Maybe doc/...
- cmr: We have a really old language FAQ.
- brson: This should replace the bad FAQs we have in the repo.
- nmatsakis: I will review it in detail. Maybe I'll add some type system rules (why &mut is not aliasable).
- brson: The answer is that you should turn this into markdown and make a PR. Do you need anything more on it today?
- cmr: No.
- brson: Awesome!

# RFC 12 tweaked variance

- nmatsakis: Need pnkfelix & nrc here. Briefly, the rule of thumb is that if you have a type parameter or a lifetime parameter that you don't actually use, it gets treated as if there is data that is reachable that uses it. So:
```
struct Foo<'a> { x: *T }
// Foo<'a> behaves roughly like &'a T
```
- nmatsakis: Unfortunately, today, we just throw away the `'a`, but that's not usually what people wanted. They usually wanted it in there. This comes up in slices from the slice iterator. Today, people insert an explicit marker:
```
struct Foo<'a> { x: *T, marker: ContravariantLifetime<'a> }
```
- nmatsakis: The downside of this approach is that people don't know what "ContravariantLifetime" means. I also don't know of any use cases that are different than that. There are two options. Current is not good. One is to report an error for unused ones. The other is to do away with the markers and add a rule that if it's not use, default to Contravariant - what it would be if there were a marker in there:
```
struct Foo<'a> {...} behaves exactly like
struct Bar<'a> { x: &'a T }
with respect to variance
Foo<'static> and use it for any Foo<'a> 
but could not use Foo<'a> where Foo<'static> is expected
```
- nmatsakis: With opt-in builtin kinds, we don't need them to opt out of anything. Could still have a lint about unused parameters...
- brson: I like that it gets rid of markers! Should have called the RFC that.
- nmatsakis: Renaming now...
- acrichto: There's a lot of feedback... 
- nmatsakis: It's hard to know if that feedback applies to our case. If there's a fallback we need for that case, we can always add an explicit marker for that. Perhaps there's something with HKT where you're forced into a lifetime parameter in order to meet an interface, but you have only static data. But, we don't have HKT. nrc has some objections, but we should follow up.
- brson: Can't make progress without nrc, right? Well, I like this.

# Assigning PRs.

- nmatsakis: I'm having trouble tracking what I'm supposed to do. People do r?, but that vanishes in about 30 seconds. In bugzilla, you assign to somebody. Maybe we should assign PRs to reviewers? And you could assign to somebody else? Do you think it would be helpful.
- jack: Can only assign to a commiter...
- nmatsakis: But we're inclined to decrease the number of people with push access...
- brson: Non-commiters also can't assign.
- jack: A bors page could list who is waiting.
- nmatsakis: I was hoping github would help.
- cmr: Phabricator would help... it's the one used at facebook. It has issue tracker, code reviewer, built-in wiki, etc.
link: http://phabricator.octayn.net/
- cmr: I have it set up to pull from our github repo & flag commits for audits. Some people use it to review code. You can submit a diff and use that for a review instead of github's interface.
- jack: We use Critic. I don't think it fixes this particular problem.
- nmatsakis: I'm upset about github, but too lazy to do anything about that.
- pcwalton: Critic's ok. Parts of it are annoying. But that seems to cover all code review tools.
- acrichto: GitHub is just death by a thousand papercuts.
- brson: It's fine for small projects, but at scale everything starts to fall over.
- nmatsakis: I just would like to have in-progress comments as they go...
- jack: You should just check out our Critic tool. Check out a Servo PR, start playing around, etc.
- nmatsakis: How well does phabricator work with github?
- cmr: Zero. I could write some scripts, though. 
- larsberg: Have to push patches to PRs instead of rebase/squash until end.
- pcwalton: That's a hard part of critic. The rebases just ruin everything (you lose commits).
- acrichto: phabricator is great for diffing between diffs.
- nmatsakis: I don't see us moving off github anyway in the short run. I have decided I shall simply assign to myself.
- acrichto: Can we sit down with GitHub? 
- larsberg: We've tried before and had people sit down with them and had no luck previously...
- brson: They have done us some favors before.
- pcwalton: Sizes of things displayed. 
- brson: And adding Rust support to various systems. 
