# Agenda 12/17/2013

- attributes with an enum (acrichto) https://github.com/mozilla/rust/pull/10937
- removing {As,Into,To}... traits (acrichto) https://github.com/mozilla/rust/pull/10967
- removing extern mod(attr = "value") (acrichto) https://github.com/mozilla/rust/pull/10696
- hierarchichal stdlib (acrichto) https://github.com/mozilla/rust/issues/9208
- return in closures (acrichto) https://github.com/mozilla/rust/pull/11024
- struct/class (acrichto)
- 'self? (pcwalton)
- extern mod -> crate? (pcwalton)
- 0.9 (brson)
- pkgid (brson)
- rename push/pop/unshift/shift et al (pnkfelix) https://github.com/mozilla/rust/issues/10852

# Status

- nmatsakis: #3511 (lifetimes of temporaries and so on)
- acrichto: libgreen/libnative, landing comm rewrite
- brson: android bot, cold
- pnkfelix: GC
- pcwalton: pointer reform, servo

# 0.9

- brson: let's try to get this out in 1st or 2nd week of jan. if you want to get something in now is the time.

# 'self

- pcwalton: it got removed, and i don't remember what i wanted to talk about with that.

# attributes with an enum

- acrichto: we got a PR for this. you'd look it up via an enum. this works great at the outer layer, but none of the inner things are enum related, so it seems like a large compiler change. we should decide if we want to change how we represent these.
- brson: this would require them all to be predefined?
- acrichto: yes.
- nmatsak: does it have a catchall for attributes that aren't yet defined?
- acrichto: i'm not a fan of this. i like the string representation. they are still represented as strings in the AST.
- brson: they are supposed to be extensible metadata and this hard codes them all.
- acrichto: we have a hardcoded list right now inside of a lint pass.
- brson: i'm not crazy about it.

# pkgid

- brson: last week we removed link and replaced it with pkgid. i was premature to r+ this because it caused disruption to downstream source code. i was wondering what the plan is to straighten this out. we had the idea to make the crate hash consistent and calculable by external tools. to do that we changed the link attribute to a pkgid string and this string is hashed for the crate hash. that way the makefiles can know what the output name is going to be. the problem is that all the existing packages got their name incorrect because they didn't have pkgid strings. all of a sudden packages have to have the name be the name of the source repo. finally, it directly ties crates to packages. so we've conflated the concept of crates an packages.
- pnkfelix: the reason we replaced the three separate fields with one field to make it really simple?
- jack: yes. extern tooling already complicated wth this and so minimalism was desired. also, the second issue there is now fixed. that just landed.
- pnkfelix: instead of being possible to extract why not use a rustc flag.
- jack: i tried to do this but was a little more involved than i was comfortable with. output names aren't known unitl phase 6 but needed here after phase 2 or even phase 1.
- nmatsakis: why do we do the weird names?
- jack: if things all end up in /usr/lib you want disambiguation. it's a tradeoff, the question is is it worth id?
- acrichto: i think it's worth it.
- brson: are we comfortable with what we've done, replacing link with pkgid?
- pcwalton: it's fine. i want it to be simple.
- brson: i don't care that much easier but it broke stuff. what about the name pkgid?
- acrichto: there's a post about this.
- jack: i agree it's bad but interrelated to all of the other things like extern mod syntax.
- brson: this stuff will interact with a future rustpkg, so hopefully whatever we do will work with that.
- pnkfelix: the pkgid string, the one advantage of link attribute is that it's forward thinking in that you can add other metadata. can we extend pkgid?
- jack: this is not very flexible because want to avoid canonicalizing things.
- acrichto: ??  "if we have rustc --print-package-name then other tools shouldn't need to do anything"
- pcwalton: there is a danger of reverse generalizing here. java decided to use reverse domains. i want to make sure whatever we do is sipmle. for example, node has one alpha numeric string and that's all you can do. they seem to get along fine, despite the obvious problems of this.
- pnkfelix: i don't care about the syntax as much (slight preference for link attributes). but we should decide whether external tooling will be able to compute this. we could do this with rustc in the future so we can canonicalize in that case. that would give us control over this even if it is trivial for now.
- pcwalton: is the hash based on types?
- jack: the symbol hash is still based on types, but crate hash is only the string
- brson: how about we add the flag ot the compile and rename this to crate_id

# extern mod name( ... )

- acricto: need ot look at it more.

# extern mod -> extern crate

- pcwalton: we've discussed this several times and there was a general feeling that extern mod is not the best name for the external crate because mod means something different. what it really does is link against the crate. we could say 'extern crate' instead.
- nmatsakis: i don't see the point in 'extern". crates are always extern.
- brson: you can imagine if we use this then the top of the source file will have crate_id and then 20 external crates.
- pcwalton: use crate is another alternative.
- jack: was also going to suggest that
- nmatsakis: use crate is not bad
- acrichto: people will expect to say `use crate foo::bar`.
- pcwalton: i wouldn't expect that.
- acrichto: but use statemtns have long paths with colons. if we put it there then people will expect to link and import at the same time.
- pnkfelix: a good error message would solve this problem
- dherman: use mod and use crate would be different
- pcwalton: you'd write `use crate foo` not `use foo`.
- acrichto: would we have the same restriction on ordering?
- pcwalton: yes
- acrichto: it seems weird that we'd be reusing the word use even if it reads well.
- pcwalton: we already did this with impl and impl for.
- nmatsakis: i don't find it confusing. we could extend it in the future.
- pcwalton: extern also had this dual use although 
- acrichto: of crate, extern crate and use crate i like use crate the best.
- nmatsakis: seems like people get confused betwteen declaring and referencing modules. do they get similarly confused about using crates from multiple locations?
- jack: we could disallow anywhere but in main crate
- pcwalton: useful for testing sometimes.
- brson: after extern mod std then use std, so what about that?
- nmatsakis: that's pushing me against that. this is like pub use but not really since that takes osmething declared elsewhere and reexports.
- pcwalton: at some poitn sugar wins over consistency.
- dherman: i never really undersrtood extern in c, but i had the idea i'm talking about linking. i like the idea of using extern in talking about crates.i feel like design question like this don't work in large groups. a few people should probe the space.
- nmatsakis: to some extent that is what we did. now we're getting feedback on it.
- brson: i don't wan tto feel like i should have cared more.


# return in closures

- acrichto: it was pointed out that after the for change we can have return in closures
- brson: people are for this?
- nmatsakis: back when a for loop was a closure it was mind boggling. now it's not so mind boggling. there is precedent both ways.
- acrichto: are there other languages that allow this?
- pcwalton: javascript
- nmatsakis: java too
- dherman: the TCP thign is not philosophical. it's a practial issue for macros. you can't take a statement and wrap it in the clsoure without screwing up what return means. exactly what niko was talking about with for loops. 
- pcwalton: break continue and return all have this problem. but break and continue generate errors and return wouldn't. philosophically generating an error is scarecely better in terms of TCP than making ti work. you're still violating TCP either way.
- dherman: error message won't make sense. return 42 when outer is unit and they won't know what to do. i'm scared aabout how non-chalant we are about these decisions. as brian was saying this is a hard call.
- pcwalton: i just feel that TCP has already been violated and other C langauges have as well. the way to respect TCP is to have break and continue have labels and that's too much.
- dherman: maybe in the long run we will find we'll want some TCP respecting lambda form that macros can use. you could add a new closure type for the macro system to use that doesn't capture return and continue.
- pcwalton: the problem with that is it requres exceptions or something like that to implement.
- dherman: i think it makes sense that the macro system is less than a lisp macro system because of the space we're in.
- pcwalton: a macro system should be able to pull expression into a clsoure and invoke that later. you can't do that in rust with perfect fidelity.
- pnkfelix: maybe a solution is to have the macro system have return-free sub-expressions.
- pcwalton: that may be a happy solution.
- dherman: this gives you nice error messages
- nmatsakis: i used to think the other way, but people have asking for it so long and i have wanted it.
- pcwalton: we're picking our battles carefully and it's a battle that may be won but won't be fought by us.
- dherman: i'm on board.
- acrichto: i'll approve it.

# option / result 

- acrichto: they have as_result and into_option and no one uses them. someone had a PR to remove all of them. what do people think?
- nmatsakis: i don't even know what they do
- acrichto: they are made for conversion from result and option. why not just have hte method declared inline with the struct.
- brson: this is straight overdesign.
- acrichto: ok. will look again and see if we can remove it.

# stdlib hierarchy

- acrichto: we have libstd and a flat namespace. it was brought up that we need to do this for 1.0 because it will change everything. there are some specific suggestion like grouping containers and pointers. do we want to go in this direction and reorganize? or do we want to leave it as is today?
- pcwalton: there's this stuttering problem that we discussed `rc::Rc` and such. we've been fighting with this for years and is one of ht reasons we capitalize type names. one data point here is that Simon has hte servo style crate has a lot of internal organization but the public api is pub use and not stuttery. that is worth considering i think. where you can keep the file organization but you don't have to name it in the public api.
- acriichto: we've done away with the top level functions in most places so we only need the type. this issue is vague so the question is more do we want to entertain the idea to change how this is done.
- pnkfelix: i like SImon's approach
- pcwalton: it's working well in servo to have less deep namespaces up front. iterators can't be done that way. you can't namespace everything under the type today.
- acrichto: some things have common names like Hashmap, but the iterator is not named. i might talk with him a little more in IRC because unless we have a pull request that's specific it's hard to say anything.
- brson: i think we'd like to have a more complete proposal.

# rename push, pop, unshift, shift

- pnkfelix: i figured erik's being doing research and i'm worried that it's pointless to keep adding examples. we have a trait system that lets us have more naming control. we don't have to make the same choices other languages made. does anyone else have an opinion?
- pcwalton: our names came from perl. shift and unshift have never been hard for me because i was/am a perlist. but it makes no sense for anyone else. i'm fine with prepend and append.
- pnkfelix: i like the fact that shift and unshift reflect the performance. i think that uniformity belongs in the trait but not necessarily on the types.
- jack: i mentioned that clojure does something like this
- nmatsakis: i think the goal is to have only one set of names instead of two sets. it's sort of in conflict with that idea.
- pnkfelix: which is more important then?
- pcwalton: i am in favor of consistency. i keep asking what is unwrap called today. i do see the point of expensive things looking expensive. it's in direct conflict.
- nmatsakis: i'd like to see a conrete proposal with different types. if i were going to make a proposal i would try for this approach for fast names and explicit names separately.
- pnkfelix: does anyone else read push_back as the acction of pushing back on someone who makes a bad suggestoin?
- dherman: i always thought it was awkward and long winded.
- pcwalton: i feel like push_back is so much more common that it's verbose. the history of the STL is that the creator hated abbreviations. that was an explicit design goal. that was in the java enterprisey days and was a product of its time.
- nmatsakis: what about eh case of a queue where the fast ends aren't hte same?
- jack: i don't remember what clojure does. mayb eit only has a fast push (conj)
- nmatsakis; append and prepend seem the most natural to me.
- pnkfelix: i'd rather have append/prepend than shift/unshift. the performance model part is secondary.
- niko: what's the opposite of append and prepend?
- pcwalton: pop and pop_front? i don't know.
- dherman: i find shorter names more appealing. my experience in javascript is that i got used to it after a while.
- brson: maybe there's not even momentum yet to change things. i still don't know what shift/unshift do. i know what push and pop do, but not what shift and unshift do.
- nmatsakis: shift takes it off? (laughing)
- jack: maybe we swap what they do to match more people's misunderstanding (laugh)
- pnkfelix: shift_onto shift_off
- dherman: *pend words are for lists instead of atoms.
- nmatsakis: what about push/pop/insert/remove and the latter 2 take an index.
- pnkfelix: i second niko's proposal.
- niko: i think that's what java does too.
- dherman: do you care abou the performance of comparing the index?
- niko: no.
- pcwalton: that's fixed by inlining it and it becomes dead code.
- dherman: my little pony: inlining is magic
- brson: out of time. that was fun!



