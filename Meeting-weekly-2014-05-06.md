# Agenda 5/6/2014

* Vec\<T> vs Box\<\[T]> vs ~\[T] (acrichto)
* Strbuf/str/chars/etc (acrichto) https://github.com/rust-lang/rfcs/pull/60
* enum keyword (acrichto) https://github.com/rust-lang/rfcs/pull/27
* module directory structure (acrichto) https://github.com/rust-lang/rfcs/pull/63
* reserved keywords (acrichto) https://github.com/mozilla/rust/issues/10293
* Box in the prelude (acrichto)
* Box\<self> (acrichto)
* unsized/type/Sized? keyword (nrc)
* ufcs rfc (nrc) https://github.com/rust-lang/rfcs/pull/4
* locally printing stickers for Rust Paris meetup; budget? (pnkfelix)
* Trait\[Bounds] (pcwalton)
* significant addresses/statics (pcwalton)
* unsafe extern C functions (#10025, pcwalton)
* reflection behind a feature gate (pcwalton)
* close "u8" -> "uint8" etc. (pcwalton)
* library principles/conventions doc (aturon)

# Attending

- acrichto, brson, aturon, pcwalton, bjz, nrc, pnkfelix, larsberg

# Status

- acrichto: I/O, timeouts, bots, libcore
- nrc: DST
- aturon: landing new Process API, working out new Task API, atomics
- brson: VPC, contracts, docs
- pcwalton: de-~
- pnkfelix: makefile hacking, polishing control flow graph rendering

# Action Items

- pcwalton: Convert to Vec\<T>
- pcwalton: Remove ~str, &str as lang item
- pcwalton: close uint8 issue
- pcwalton: gate reflection
- acrchto: #10025

# Vec\<T> vs Box\<\[T]> vs ~\[T]

- acrichto: Just talking about vectors, today, we have Vec, ~\[], and slices. ~ is being removed (per the accepted RFC). Seems inconsistent to leave ~\[]. Natural extension is `box [T]`, which is really bad for My First Vector Type. This is pretty concrete evidence that maintaining owned slices for now is not a good idea; almost all internal apis should be Vec.
- pcwalton: I agree.
- acrichto: This is influenced by the string decision, because Vec\<u8> -> strings. How does this sit with everyone?
- nrc: The overhead of an extra word is insignificant, because we expect Vec to be a lot of data. Is that fair? We have SmallVec for when it matters.
- acrichto: box \[T] will be supported, but it will not be common. If you build a library, you will probably return Vec\<T> instead. As a consumer, you can put it in Box\<\[T]>, which is not growable, and will not be very common.
- pnkfelix: What about Vec\<Vec\<T>>? A Box\<Box\<\[T]>> and coersion to borrowed slices... 
- brson: You can convert them, right?
- pnkfelix: Yes, you can convert, but not coerse because they have a differently-sized representation. It's not a big deal to me.
- acrichto: Definitely a legit concern, but probably not a big enough deal to prevent the migration.
- brson: I agree. Saying "use Vec for vector types" is clear. This is basically just messaging, right?
- acrichto: Probably just something to the mailing list. Also, code-wise, we need to change the standard library to stop using ~\[].
- brson: aturon, you were talking about conventions? So, it would land in that document...
- pcwalton: Can I just remove Box\<\[T]> transitionally? I originally implemented Box\<\[T]>... but I don't want ~Box\<T>. I want to remove ~ from the language as soon as possible.
- brson: Can we deprecate it to make the transition easier? Or should we just remove it?
- acrichto: I think people will follow the standard library. Once we don't take it, people will move to not use ~ anymore. It would be nice to mark it as deprecated for a little bit...
- pcwalton: I already didn't do that for ~, but I can add it back.
- nrc: Is ~\[] used more than ~T?
- acrichto: It was THE vector, so yes. 
- pcwalton: I don't agree. This is a major policy shift, and I don't think we need an obsolete for something like that...
- brson: I'd like to give users more help. We don't have to.
- pcwalton: So actually continue to accept code that has ~ for a while?
- brson: Maybe?
- pcwalton: Now's not the right time to be doing that.
- pnkfelix: It's nice for showing we're able to do gradual deprecation, but not a huge deal for this particular change.
- brson: Not a lot of support for this...
- pcwalton: Seems like a slippery slope; once we add more deprecation stuff, it may increase friction, etc.
- pnkfelix: Maybe deprecate with "will disappear in 0.12?"
- brson: I was just thinking of months.
- pcwalton: But what if we want to have bitwise-not? This prevents us from moving quickly if we leave it deprecated.
- jack: Maybe next release we'll start doing obsoleting.
- pcwalton: It's a lot of infrastructural work... we'd have to have a special table to specify if it was old or new syntax in order for the lint to trigger...
- brson: Put it in the parser. You can't turn off the warning or turn it into an error. It's just a heads-up.
- pcwalton: Having no way to make it an error means we could end up with us not generating errors on the ~, and we'd keep growing more ~.
- brson: Maybe let's not start this battle and forget that I brought up the obsoleting thing. Also, who's responsible for the library conversion?
- pcwalton: I'll do it. I've got the patch to remove ~ ready; just have a day or two to do vectors and strings.

# Strbuf/str/chars/etc

- acrichto: The name of one string type is easy, but we need two. We don't have a convenient syntax for slices like we do with vectors. There are two prevailing ideas: the main (owned) string would be Vec\<u8>. Internally, that's the main string type. The Box\<str> is almost never used. The questions are: what do we call it, and what do we call the slice types? String (capital S) for the owned string. And for the Box type, "chars" or "str". Within each, we could capitalize or not. chars or str would operate exactly as today. &chars or &str is the slice type. Thoughts?
- pcwalton: Slight community preference for string vs. str. I have no opinion.
- pnkfelix: My reading was that it was comparining String vs. Str. For slice type, they seemed OK with lower-s str.
- acrichto: The biggest objection is that if the owned is capital-S, then will people be confused?
- pnkfelix: Capital-S for StrBuf...
- acrichto: If we go with chars, then the owned string could be str, possibly....
- nrc: The two names should be different and not be distinguished by abbreviation or capitalization. I don't care which we choose, but they should be different.
- acrichto: So you're opposed to String and Str?
- nrc: Yes.
- pcwalton: Community prefers String and str. Failing that, Str and chars. Some people had visceral reactions to chars....
- pnkfelix: I was trying to understand Simon's post, but couldn't see if he wanted utf8 or didn't want utf8.
- pcwalton: People expected chars to be an array of characters or an iterator over characters. objected to it not being a flat array of characters. I don't agree with this...
- pnkfelix: Yeah, we're removing indexing anyway, so it doesn't really matter. (the current indexing returns a part of a character, not a char).
- pcwalton: You could have an index that does the Right Thing...
- nrc: With Str and chars, the growable string would be a Str. Slice would be &chars. And Box\<chars> would be something you shouldn't use...
- pcwalton: Question of whether chars should be built-in or a langitem. nmatsakis and I prefer a langitem, though acrichto prefers builtins.
- acrichto: I like not having to define this in core and not have to have a langitem to write a string. I can have an integer literal without a langitem, etc. But I'd rather debate naming.
- pcwalton: Affects naming - langitems are capitalized, and non-langitems are lowercase. nmatsakis also pointed out that with langitems the compiler is simpler because you can hand off the support to the stuff that already does this for user-defined types.
- brson: Seems like langitem consensus.
- pcwalton: Should it be a DST langitem?
- acrichto: Yes.
- pcwalton: So it is &chars or &Str, regardless... can we just call it chars? We're already using that...
- brson: I think you should make the change, pcwalton, but we should leave the naming question open for another week, at least. It doesn't feel like we're ready to decide.
- acrichto: Remove ~str and then we'll have to move forward. Algorithmically, we're going to remove something...
- pcwalton: Can we call the new thing chars?
- acrichto: Remove ~str. Call everything StrBuf.
- pcwalton: But we wanted a langitem... what should it be called? I'd like to implement it as chars and leave the naming open. Big perl script to replace.
- nrc: StrBuf and chars?
- huon: Can we implement w/o DST?
- pcwalton: Yes.
- acrichto: Only for owned. Rc\<chars> would not work.
- pcwalton: Only the borrowed case, which is the important one anyway.
- acrichto: I'm worried about big changes like this where we were trying to make incremental progress but others thing of it as signaling the One True Path.
- pcwalton: The alternative is doing nothing.
- acrichto: Can remove ~str without renaming str...
- pcwalton: We want to make it a langitem - why wait?
- brson: It's just a worry about setting poor expectations.
- pcwalton: What's the concrete problem here? Worst-case, somebody doesn't like the name chars and proposes something better, which seems fine.
- pnkfelix: The worst case is we go from str to chars, and then go from chars to utf8, and keep making massive renaming/file-touching for no real reason...
- pcwalton: Making it a lang item has compiler support that has to happen.
- brson: Can we let the current string & new string impls survive together? So that our users don't have to do the rename twice?
- pcwalton: Maybe, but I'm not happy about it.
- huon: Could we just have the lang item lower-case str until we decide?
- brson: We'd at least not break users twice.
- pcwalton: OK. But when are we going to make this decision? Is there some data we're waiting for?
- brson: What's stopping us is simply that we don't agree.
- pcwalton: What are the positions? I don't care.
- acrichto: I like String and Str. nrc and brson don't. pnkfelix seems to like StrBuf.
- nrc: The RFC thread is total chaos. I'm trying to follow the comments, but I can't really sort it out. It would be nice to interpret the data from it and try to clarify the constraints from various people to make a better decision.
- acrichto: I think there are no more than two people behind any given proposal.
- nrc: It's not a vote, but it'd be nice to at least understand the positions and naming choices better.
- acrichto: Well, we can at least remove ~str for now. And then we're set except for naming. So at least it's SOME progress on this topic...
- brson: We can do a lot in the next week without making the naming decision. Let's move on.
- pcwalton. Wait! Can I implement &str as a lang item?
- acrichto: If it's still named str, that's fine. But renaming without knowing we will stick with the rename is a bad idea.
- brson: Yes.

# unsized/type/Sized? keywords

- nrc: This is the syntax for marking a type parameter as accepting DSTs as well as regular (statically sized types). Context on the naming decision is that the easiest thing is you can't just use a regular bound because accepting unsized as well as sized lets you do more, whereas bounds let you do less. The RIGHT thing to do is that most methods accept both and you narrow it down, but it would be way too noisy in practice. The next is proposal is unsized. The later one is type, which on a type variable denotes it accepts unsized & sized types. Even though that's slightly misleading, that's one alternative. The proposal that niko, felix, and I agree on is if we implicitly accept that all type variables have a bound (Sized), then you can say that a type variable doesn't have it by using that bound with a ? operator before that variable. Simplest case would still be T. You could then add `T:Bounds`, but make them optional by `T:Bounds?`. 
compare:
```rust
  fn foo<T>(x: &T) { ... } // .. T is implicitly given the Sized bound here.
  fn foo<Sized? T>(x: &T) { ... } // T is now marked as *not* having the implicit Sized bound

fn foo<T: Send> { ... }
fn foo<Sized? T: Send> { ... }
fn foo<Sized+Foo? T: Send+Freeze> { ... }
fn foo<Sized? T: Send> { ... }
fn foo<Sized? T: Sized> { ... } // lint warning: redundant (why would you say Sized? and then add Sized bound  --- potential answer: *macros*)
```
hypothetical (assumes existence of `Unsized`, and a use-case for this feature):
```rust
  fn foo<Sized? T: Unsized>(x: &T) { ... } // T is now marked as *must* being Unsized.
  fn foo<T:Unsized>(x: &T) { ... } // This would be an error, since you cannot have a T that is both Sized (which is an implicitly bound on T) and Unsized 
```

alternatively
```
fn foo<unsized T> { ... }
fn foo<unsized T: Send> { ... }
```
- brson: The question mark means it is unsized?
- nrc: No, that it might be sized. Without the `?`, then it's definitely sized. If you write `Sized?`, it MIGHT be sized.
- brson: And you can remove the ? and it'll be definitely unsized?
- pnkfelix: Could add it to the compiler; it's in the examples above...
- brson: No use case? And this scheme would apply to other future hypothetical bounds?
- pnkfelix: Yes, it's the first proposal that scales to other things in the future.
- acrichto: Why is it on the left instead of the right?
- pnkfelix: It's not a bound... maybe if we had an upside-down-?, then I'd be fine with it. I think of it as an infix operator between the type and the bound parameter.
- brson: Looks great to me. Anybody uncomfortable?
- acrichto: What about multiple ?-bounds?
- brson: There is an example: +.
- pnkfelix: Not sure if that means could be Sized+Foo?...
- acrichto: Sized is the only thing allowed before ?, yes?
- nrc: Yes. Has to be something that is implicitly allowed. I recommend not allowing multiple bounds before the ? for now, to ensure that we have that freedom in the future.
- pcwalton: lgtm.
- acrichto: is there a DST RFC?
- brson: I think we should land this; we have announced DST broadly and widely.

# Box in the prelude 

- pcwalton: I assume no sane people object, right? Good.

# Box\<self> 

- pcwalton: In my patch, you can write a fully-explicit form:
```
impl Widget {
    fn foo(self: Smaht<Widget>, ...) {}
    fn bar(self: &Widget, ...) {}
}
```
- pcwalton: Do we want the following?
```
impl Widget {
    fn foo(Smaht<self>, ...) {}
}
```
- acrichto: Not really. You're declaring a variable where a type parameter should go...
- pcwalton: That's how &self works now.
- acrichto: What if it wasn't the first parameter?
- pcwalton: Don't do that. Can't use the sugar, then.
- brson: You do like the sugar, pcwalton?
- pcwalton: Yes. Though there were objections, so I was implementing the long form. My patch also does the explicit sugar for other methods.
- nrc: I thought we discussed this at the workweek...
- pnkfelix: nmatsakis has thoughts on this topic, too. I believe he wanted to try to get rid of self being so special, so that any method with that type as the first parameter would just work.
- pcwalton: I did not understand that anyone wanted to move away from sugar. Still wanted &self for methods.
- acrichto: Looking at the meeting notes, the biggest was that the sugary form of &self should be self:Self... nothing related to smart pointers there. We did say we should keep sugar during the workweek, but nothing about w.r.t. smart pointers.
- pcwalton: I think we need nmatsakis here. I'm implementing the fully-explicit form for right now.
- acrichto: Just for &self and Box\<self>?
- pcwalton: And by-value self.
- brson: Sounds good.

# locally printing stickers for Rust Paris meetup; budget?

- pnkfelix: How can I get stickers? Do I print them locally? Our local person got declined as a rep and denied able to get stickers through Mozilla, and wants to be reimbursed...
- azita: How many?
- pnkfelix: A hundred or so. But stickers seem very expensive in Paris.
- azita: They ordered stickers might not be identical. We have some in the room that we can send and we will order more later.
- brson: Tell him to send me his address and I will send them immediately.
- pnkfelix: I will have Axle get in touch with you.

# close "u8" -> "uint8" etc. 

- pcwalton: I shall close this.
- acrichto: Close? Merge? What's the plan here?
- pcwalton: I don't care.
- brson: Let's close.

# Reflection behind feature gate

- pcwalton: Reflection behind feature gate. Is that OK? Or experimental?
- brson: Just, what do we do with the ? formatter? Maybe put that behind a feature gate?
- acrichto: Tons of compiler support for :?. In theory, the compiler could interject calls to experimental code...
- pcwalton: Only if you type :?.
- pnkfelix-dog: very excited!
- brson: At expansion-time, do we know what features are enabled?
- pcwalton: We could set a taint flag in the crate for if the experimental stuff is being used.
- acrichto: The other option is for it to be like libdebug, where if you link to the library you get it and if not you don't. But I think we can't do that.
- huonw: Wouldn't the flag mean that all standard libs would have to avoid it?
- acrichto: I think they do - I removed it a while ago.
- brson: Any objections to gating this?
- pnkfelix: Not at this point.
- acrichto: Since deriving(Show), this has been less necessary.
- pnkfelix: It doesn't work for Vec, so it was useless to me now.
- nrc: I prefer to see the address!
- pnkfelix: I disagree!

# unsafe extern C functions

- pcwalton: The unsafeness of a function could be determined by the unsafe effect instead of the ABI. Might want callbacks from C that are extern C but not unsafe to avoid the effect... So, now, unsafe would never consider ABIs; only whether the function is marked unsafe. Also, all functions marked in an extern C block always have an unsafe effect.
- huon: Same as what we have now?
- acrichto: An extern fn foo would be different:
```
extern fn foo () {}
fn main() {
    foo();
}
let foo: extern unsafe fn();
```
- brson: The problem is that unsafe things can sometimes be treated as safe, and sometimes not - the compiler is not consistent.
- acrichto: I'm lost.
- huon: You can't write `unsafe extern fn` - it's entirely syntax-directed.
- pcwalton: Does anybody object to `unsafe extern fn foo`? 
- acrichto: I agree; I think it's just being able to write `unsafe extern fn`.
- pcwalton: kmc pointed out that what you want is `unsafe extern C fn`, so that you have a function that's unsafe that you trust C to call but not your regular Rust code.
- huon: `unsafe` before or after `extern`?
- pcwalton: Before.
- acrichto: A valid type today is `extern unsafe fn`. Those should remain the same. But I don't see why we wouldn't allow extern fns to be unsafe. That is an entirely separate effect.
- pcwalton: So, we will allow `extern unsafe fn` to be parsed.
- acrichto: What order?
- pcwalton: I prefer `unsafe extern` but really just want something required to make it greppable.
- brson: I agree.
- acricho: I have a burning desire to do this.
