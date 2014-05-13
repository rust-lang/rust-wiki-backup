# Agenda 5/13/2014

* String/&str (acrichto) https://github.com/rust-lang/rfcs/pull/60
* module directory structure (acrichto) https://github.com/rust-lang/rfcs/pull/63
* reserved keywords (acrichto) https://github.com/mozilla/rust/issues/10293
* blocks in constants (acrichto) https://github.com/rust-lang/rfcs/pull/71
* ufcs rfc (nrc) https://github.com/rust-lang/rfcs/pull/4
* Trait[Bounds] (pcwalton)
* significant addresses/statics (pcwalton)
* attribute style (cmr) https://github.com/mozilla/rust/issues/11193
* GDB version on bots https://github.com/mozilla/rust/pull/14031
* quickcheck (huon) https://github.com/mozilla/rust/pull/14100
* let mut (nmatsakis) 
* linux 32bit abi issues (jack) https://github.com/mozilla/rust/issues/14177

# Attending

larsberg, acrichto, luqman, cmr, pnkfelix, pcwalton, brson, aturon, bjz, jack, azita

# Status

- brson: administrative, minor things
- acrichto: libcore refactorings, 32-bit bug, pretty printing improvements
- pcwalton: de-~str
- aturon: library design docs, iterable, task/taskbuilder, other small tickets

# Action items
- acrichto: follow up with nrc and nmatsakis about String/&str
- acrichto: merge RFC #63
- acrichto: merge RFC #71 


# Interns

- brson: We have two new interns who started yesterday! Luqman and cmr. 

# String/&str (acrichto) https://github.com/rust-lang/rfcs/pull/60

- acrichto: Consensus on the bug is in favor of String and str, renaming StrBuf to String. nrc is very opposed to that, but is not here. brson also had concerns?
- brson: I don't particularly like the solution, but it is a pretty easy one to use. Does anyone else have a strong opinion?
- pnkfelix: In favor of String and str, as is niko. I'm sympathetic to nrc's issues that you should not have two distinct concepts use a name that is an abbreviation of the other. However, I feel that String is a sort-a subtype of str. I don't know if that's a fully convincing argument.
- brson: We should at least talk to nrc and nmatsakis.
- aturon: I prefer StrBuf, but don't have strong opinions.
- brson: Can you follow up with nrc and nmatsakis to try to sell them on it?
- acrichto: Sure.

# Trait[Bounds]

- pcwalton: There is an ambiguity in the grammar, around trait : bounds. Example:
```
    fn foo(f: |x: int|){}
    fn foo(f: |Trait:Send|){}
```
- pcwalton: In closure syntax, you can optionally provide names for the formals. The problem is that you don't have to, but there is an ambiguity whether Trait:Send is a name and type or type with bound. Niko proposed changing it to:
```
    Trait[Send]
    Trait<T, U, V>[Send + Share]
    proc(T, U, V)[Send + Share] -> int
    |T, U, V|[Send + Share] -> int
    
```
- acrichto: Could also either require or disallow the argument name.
- jack: Or Trait:Send is always argument:type and if you have Trait:Send, you have to name it.
- pcwalton: It's useful to include the names if your closure types have lots of arguments, but pointless if they don't. Also follows C precedent. I don't like the second because it's an ambiguity and would be nice to be consistent.
- nmatsakis: I'm in agreement with those two arguments. I'd like to also toss out the idea of using the + operator. It could generalize to have an arbitrary set of traits. So you could have:
```
- ~(Trait+Send), ~(Foo+Clone+Send), ~(Trait+'a)
```
- nmatsakis: kinda weird looking, especially with region bounds, but it's something we were talking about to free up the :.
- <mumble mumble>
- pcwalton: I like the thing with the +.
- nmatsakis: You could make a type alias for multiple traits. With DST, that works for multiple object types. It would be something like:
```
Trait[Send+Share]
- Could make type aliases for compounds: `type Stream<T> = Read<T> + Write<T>`
```
- pnkfelix: Could get rid of Stream.
- bjz: Can you type `as` and then have those added types afterwards? Or is that an ambiguity?
- nmatsakis: `as` takes a type grammar. Would need a paren. That's a good question we have to answer. It hurts my eyes to not see parens around types under pointer sigils. This is fine:
```
Box<Trait1+Send>
foo as uint + 3
```
- nmatsakis: Could have some subset of the type grammar available after the as, in support of the `foo` above. At worst, you require parens...
- acrichto: I've added some bounds up by the `Trait[Send]`. With +, how would it extend to procedures & closures? And how do you know what the grouping is and precedence of -> vs. +?
- pcwalton: What would it look like with + and procs and closures?
- nmatsakis: you can extend it by saying:

```
   |T| -> int + Send
   (|T| -> int) + Send
   
```

- pnkfelix: Is it that bad with the parens around the function types?
- pcwalton: The + there looks strange.
- nmatsakis: I think closures look strange. In an unboxed world, though, I think that closures might not be as much of a thing.
- pcwalton: I'm skeptical that we can make a big closure change for 1.0.
- nmatsakis: Maybe a langitem, like we did with Box. 
- pcwalton: That would break all rust code in existence. We need to have this discussion.
- nmatsakis: Agreed, but not this second.
- brson: We have a parser ambiguity around trait bounds. Do we have a solution?
- pcwalton: No.
- nmatsakis: There are two solutions here, with their merits...
- acrichto: Could also forbid or require naming arguments. Because some of these syntaxes for closures & procedures are really weird.
- nmatsakis: Could keep the : with procedures....
- pcwalton: I prefer the + for Traits and keeping procedures & closures the way they are.
- nmatsakis: I prefer that too. 
- pnkfelix: Can say that the closure syntax is just sugar.
- acrichto: Can I add arbitrary traits together?
- nmatsakis: I see no reason we couldn't implement that if we wanted to. But I would make it only do exactly what you can do today for now.
- brson: What is the solution?
- nmatsakis: I'll write up an RFC for this.

```
solution: Box<Trait + Send>, &(Trait + Send), leave closures and procs the way they are.
```

- brson: So, is the first name in this list special?
- nmatsakis: It would be today, depending on how general we make the code. It doesn't have to be. There must be exactly 1 normal trait and the rest must be built-in bounds. But it would be easier if it had to be first.
- brson: Everything after that (needs to be casted?)... 
- nmatsakis: I think we would say for now there must be one trait that is user-defined and the rest can be built-in bounds. I don't think it matters if that trait comes first, but it would make the code easier. Post-1.0, we could generalize to any number of traits. And the order should not matter, since + is commutative.
- brson: All satisfied? 

# attributes and spaces

- cmr: Need to decide on what style we like better. Add a space around the = sign in attributes, or no? Do we care?
- nmatsakis: you thought this was SMALL?
- pcwalton: I like not using spaces, but don't care.
- pnkfelix: Don't care.
- brson: formatting... 
- pcwalton: Would be easier to implement in the pretty printer with spaces. I just don't because of too much bash scripting, where you are forced not to write spaces.
- brson: Spaces=good. Ok, cmr?

# gdb

- acrichto: In references to libprobe, it requires gdb 7.5 on Linux.
- brson: We're not even thinking... libprobe. Let's move on.

# quickcheck.

- acrichto: Do we want to merge it or not?
- brson: Should we wait for huon? Or talk about it now?
- cmr: Anyone against merging it?
- pcwalton: I'm lukewarm. Maybe wait to see if the community really adopts it. It has a lot of traits and machinery. As a straight port from Haskell, it may not be a match for Rusty style.
- brson: Issue here is not super accessible if it's not in mainline.
- pcwalton: Regex is different because if you don't have it as a language, you're a toy. But for quickcheck, we already have a unit testing library.
- acrichto: As a procedure, should we talk about how we decide on new libraries in the distribution? or just make a quickheck decision?
- nmatsakis: Latter. 
- brson: Not ready to discuss policy. acrichto, can you follow up?

# let mut

- nmatsakis: Should I continue talking about changes here more widely, or keep iterating internally 
first?
- pcwalton: The community would love to see your RFC. I think you should make your case. But I would be surprised if it does not result in uproar. you should still make your case.
- brson: Indepenent... naming... same thing?
- nmatsakis: Is this the same as the rename of &mut? Different things, but not orthogonal. 
- brson: Will you present both?
- nmatsakis: I want to argue for removing the `mut` keyword entirely.

# 32-bit abi breakage

- jack: Been trying to port Servo to 32-bit linux. Had to port spidermonkey, but after I got it compiling, there's an ABI issue on 32-bit linux or arm or both. The gist is that there's a 64-bit sized union value in the spidermonkey bindings called jsval. It was a u64, but caused an ARM 32 bit problem, so it became a struct wrapping one, but now it's broken on linux. I'm not sure what the ABI should be or what should be done, but we generate incorrect code. If you put a number in the struct and call via C FFI, Rust gets a different number
- brson: Fix soon/urgent?
- jack: Not amazingly urgent, but at least a workaround would be good. It requires rust-mozjs. An actual minimal library did not exhibit the problem. 
- brson: Could use Regehr's C-Reduce to rip everything out of mozjs.
- jack: I'll have to watch his talk!
- nmatsakis: I think It Is Known that the 32-bit ABI is fragile. It has gotten little attention.
- cmr: Unions are a landmine, too.
- nmatsakis: oh no! unions!
- luqman: On 32-bit, an option<T> would not get wrapped. I had some fixes that change them to pointers instead that fixes some of the problems with the 32-bit ABI conventions.
- brson: That may be separate from this particular issue that jack is running into. Can we make any commitment to investigating it?
- jack: It would be nice to at least get some confirmation and maybe help with a workaround.
- pnkfelix: I will do so.

# Optimizing structs to pointers

- brson: luqman brought up fixing nullable options. The ABI turns it into structs. Luqman has a fix for the option case, but it seems like this will come up for boxes of structs. Is there a more structured solution?
- acrichto: For box to be a nullable pointer, there will have to be an attribute. If I know there's a nullable pointer and it's a word, then it's reasonable to just emit it. There has to be some way of saying that a pointer is non-nullable.
- nmatsakis: What's the problem?
- luqman: On 32-bit, we want an Option<extern fn> to be a pointer, but it gets passed as a struct. But that's not what C code does. So instead, I pass it as just a pointer, for the nullable pointer case in the ADT. Then it doesn't wrap it in the struct.
- nmatsakis: So it was already recognizing the pattern and you changed what it expands to? Seems good to me. Separate from user-defined structs. This works for more than option, IIRC, looks for any enum with non variant.
- luqman: If it's two variants and one has a filed that is non-nullable, it uses that as a discriminante.
- acrichto: Result<(), ~T> is similar.
- luqman: Yes.
- brson: So we're good?
- nmatsakis: yes.

# module structure - RFC #63

- acrichto: When we probe for files via `mod foo` we only look for foo.rs if the person is the owner. So, `mod foo` in lib.rs or mod.rs are current directory, otherwise you only search the recursive directories. lots of confusion when people duplicate modules all over the place instead of `use`'ing something. Seems like a natural extension and I think we should merge this.
- pcwalton: I agree. On /r/rust, people get really confused by `mod` and messed up by crates vs. modules. Causes lots of pain.
- acrichto: This just codifies what we do today.
- nmatsakis: Just gives better errors for people who make a mistake?
- pcwalton: Makes the system a little less flexible. 
- nmatsakis: You can use path. It's just more rigid about following the pattern if you don't say anything. I thought the behavior of this RFC was what we already did. So it makes sense.
- brson: acricho, you'll merge it?
- acricho: Yes.

# significant addresses/statics

- acricho: A static that... we talked about this at the workweek.
- brson: Everything is insignificant. But you can say inline never.
- acrichto: And a static mut always has a significant address.
- pcwalton: Are people OK with this?
- nmatsakis: Sounds good.
- acrichto: Big enough for an RFC?
- brson: How close are we to already doing this?
- acrichto: We inline kinda what we can...
- nmatsakis: We're ill-defined today.
- brson: Just do it.
- nmatsakis: Agreed.
- brson: Should have an issue on it, though.
- acrichto: Issue #8958
- bjz: Document it in the manual?
- pcwalton: Somewhere, in something nobody will read...

# reserving useful future keywords  10293

- acrichto: Do we want to start reserving them?
- nmatsakis: Even though I opened it, no.
- brson: I have some pet keywords I care about. What about you, niko?
- nmatsakis: We should be selective. Not just grab every keyword.
- pnkfelix: How about language versioning?
- acrichto: There's an issue open for that. We need it regardless for 1.0.
- nmatsakis: All that says is that we will version keywords, right?
- acrichto: If you could conditionally include a module depending on a language version, maybe that would solve this problem? Language versions may be one way to solve this.
- nmatsakis: We don't need to jump into this quite yet.
- acrichto: If we don't reserve them now, it's harder to be backwards compatible.
- nmatsakis: Java has paved the way for this. So if you don't opt in to the new version, you just get a warning.
- brson: We do have a precedent for it... we don't want do that anymore?
- nmatsakis: We should go over our reserved list to make sure it's what we really want it to be. Maybe now?
- pnkfelix: thursday.
- acrichto: There are ten right now.
```
alignof, be, offsetof, priv, pure, sizeof, typeof, unsized, yield, do
```
- nmatsakis: They all seem reasonable. Least necessary are priv and unsized.
- brson: We don't have any ideas that need unsized at this point, right.
- nmatsakis: Yeah, we have a different path forward.
- acrichto: Hold off on this until we have an idea what lang. versioning looks like?
- nmatsakis: What do we do? Vote? I don't want to add every keyword ever, though that document was a helpful contribution.
- brson: I have no suggestion.
- acrichto: Language versioning is on the 1.0 milestone, so I think we should wait. Worst case, we could add them all in a mad rush right before 1.0 and then remove them afterwards.

# allowing blocks and constant RFC 71

- acrichto: A block can have a bunch of items and a tail expression. It would allow regular expressions in statics. Seems like a nice way to scope things, etc.
- nmatsakis: I have no problem with that.
- pcwalton: Seems reasonable.
- brson: Hard to argue against it.
- acrichto: Not a large change to the language. We already accept this grammar; just changes what we can codegen.
- brson: Objections? acrichto: do it.
