## Agenda 9/17/13
* welcome alex (brson)
* rust weekly meeting attendance (brson)
* rustpkg: where to store cached source code checked out from git? (tjc)
* rustpkg: what can appear in RUST_PATH? (tjc)
* * Should https://github.com/mozilla/rust/pull/8831 become the default behavior?
* * https://github.com/mozilla/rust/issues/8520 is related
* putting macro_rules behind a flag (pcwalton)
* Local::<for Trait>::take() (pcwalton)
* Sized trait vs. "unsized" prefix (pcwalton)
* drop trait destructuring idea (pcwalton)
* default field values for structs (pnkfelix) https://github.com/mozilla/rust/issues/6973#issuecomment-24137545
* mingw upgrade (brson) (https://mail.mozilla.org/pipermail/rust-dev/2013-September/005533.html)
* whitespace clarifications (jack) https://github.com/mozilla/rust/issues/7946
* Lint reasons (tjc) https://github.com/mozilla/rust/pull/9025
* rustdoc ng for 0.8? (brson)
* OS X 10.9 (brson)
* '\0' (brson)
* '&mut self' in Drop (brson)
* crypto (pcwalton)

## Attending
acrichto, brson, jack, tjc, felix, azita, lbergstrom, pcwalton

## Points of order
- brson: this meeting is optional for Servo folks. you're welcome to skip if you like.
- jack: we'll send one delegate for a while
- brson: etherpad is a new location. this is what servo does because you don't have to figure out the date every week.
- brson: corey asked to be able to dial in to this conference. i gave him the number and we'll see how it goes.

## cached source code
- tjc: currently it puts it under src  of whatever workspace it's using. when i mentioned to jack he thought src should be sacred and checked out src should go somewhere else. the real question is there are two different scenarios. one is when you're using a lib that you'll never modify and that situation it seems godo to have it stashed out of the way. for servo the deps also get modified so it makes sense to put it into src. we also have a related case where we should make the src readonly so that you don't accidentally overwite it. right now there's no way to intiiate which one to use.
- felix: command-line flag?
- tjc: we're trying to avoid that as much as possible
- felix: seems reasonable to assume default of non library developer
- jack: the risk is that people edit the packages that they have downloaded which were supposed to be dependencies
- tjc: we could make it read-only
- jack: erlang and closure package managers put the dependencies somewhere else. in closure, if you want to edit them, you make a symlink to a subdirectory called checkout. in erlang, there's no special workflow for it and just looks to see if the package is already copied locally up a directory. 
- tjc: so if we had the sources locally and the user updated them, rustpkg would not update them. 
- jack: according to dherman, the node package manager works similarly through a config file.
- brson: for servo developers who always want to be editing all of them all the time, is there a role for the package manager?
- jack: we could also satisfy that use case. do people think that this (develop-mode) should not be the default?
- tjc: if it's a per-package thing, we would need to make it specific to individual packages
- felix: how hard is it to move packages that you depend on in the source tree?
- tjc: rustpkg should just find it
- jack: the easiest solution is rustpkg init having a small file in there that tells you which repos should be local and which to hide away. if you hide and set read-only everything that you don't call out, that would probably be the safest and would just work for most users
- tjc: are the two most common use cases "everything develop mode" and "nothing develop mode"? Seems hard to move them around.
- jack: I think the rustpkg way to do things is just to have the workspace directory be before the src directory. Then it will just pick it up if you move the file. The copy in src/ will be picked up first (which is where you would move it if you want to edit it) and otherwise the top-level version is picked up
- tjc: so then we only check dependencies for the ones in workspace, but not in the src/ directory. Seems ok.

## macro rules behind a flag
- pcwalton: we will not get to backwards-compatible macros without a lot of work or maintaining lots of legacy stuff. Notably:
* no hygiene for many types of bindings
* don't resolve item identifiers in the correct place
* don't have a scoping story (namespaces vs. macro-global-namespace)
* macro import/export don't work
* parsing bugs
* not possible to define programmatic macros outside the compiler (not back-compat issue). could maybe extend macro-rules to handle this case
- pcwalton: so, we'd be supporting lots of legacy if it's backwards-compatible. proposal is to require that macro-rules have a special crate attribute that allows you to use them. Time to think about Haskell-style definitions of language extensions used?
- tjc: in Haskell the language extension thing works pretty well. The problem is that many are closely related so you end up always using them together. Ours are probably more separate/modular, though.
- kmc: In Haskell, they're coding to a more standardized langauge. Since we're not nailed down yet, we can be more proactive about moving extensions into the core language and not having 20 directives
- pcwalton: only make it an extension if it's something we're planning to change. We don't need a committee.
- kmc: would this supplant some of the -z flags?
- pcwalton: yes. moving them out would also let them be per-crate instead of -z flags, which are for the whole invocation. Can't use the word lang because it's in use. "feature"?
- kmc: module-level attribute?
- pcwalton: crate or module.
- kmc: finer-grained control may be better. Perhaps syntactic per-module and type system per-crate, since we do typechecking at a crate level?
- pcwalton: propagate to submodules? I'd hope yes because otherwise you will say feature at the beginning of every module
- kmc: child modules in the same file but not external files?
- pcwalton: no such notion
- kmc: nice to look at a single .rs file and know what features are in use in that file
- pcwalton: question is if people sprinkle macros throughout their code or put them in a macros.rs file
- kmc: in servo, we usually have them local
- brson: macroescape? It means to "push the macro up a level"
- pcwalton: it's not about using the macro, just about defining them, so we should be OK here.
- pcwalton: I will make it propagate down to child modules to err on the side of not annoying people unless there are strong objections.
- brson: I prefer crate-level to support future type-level features
- pcwalton: sounds good

## number 6973  default arguments and keyword values
- felix: the struct should be good enough. But there's one use case where you have some arguments in your function required and some default, as we can't express that well now. Even if we make all the args a struct and make default values, you still then have to come up with a sensible default for all of them.
Proposal is to allow them to express it through an expression in the struct syntax by allowing them to put const expressions in there so that you could then leave that field out and the compiler would plug in that expression. The folks pushing the bug seemed to like this.
- kmc: like the feature. still doesn't handle putting required paramaters?
- felix: don't have a default / const expression
- kmc: aha, so then it's a compiler error. good.
- acrichto: seems odd that adding a field would not require you to add that everywhere else that you create it
- felix: but that's why there's a default in the struct - users shouldn't need to think about it. This feature is mainly for the narrow case for packaged-up sets of arguments.
- kmc: common C pattern is an ops struct in the linux kernel with 20 function pointers with default behaviors. create one using c99 syntax and get a NULL for all the others. Avoids horrible churn if someone adds another pointer.
- pcwalton: that case is subsumed by traits and default methods in rust. that's also what Go does - if you leave off the field you get a zero value, but that seems weird
- kmc: also don't like that, but it's nice to give a default value explicitly
- felix: do we limit the default expressions to const expressions? You could imagine an extension. Is there a use case for this? 
- tjc: limiting to constants seems good, and people could complain if they have good scenarios
- pcwalton: could have a more principled version of Go with a zero trait and invoke that for fields if they are omitted. But in that design you're missing out on when you wanted the compiler to tell you about all the places you were supposed to update.
- felix: once we have the const expressions in place, you could have a new deriving form that implements it
- pcwalton: zero is a runtime call. maybe associated constants, if we had them
- kmc: the other problem is that they might not be type-directed. if you want a different default than zero, you need to create a newtype, create the trait, etc.
- pcwalton: Go just requires you to love zero as a default.
- felix: will prototype this
- brson: I object! we've survived a long time without this. can it wait?
- pcwalton: fine as rust 1.1
- felix: good. we can say we like it, but it's not happening for 1.0

## language features creeping in without discussions
- brson: drop trait changed this week. no consensus. there is more discussion to be had on it. We need to stop the community from making changes without that discussion happening on language features.
- tjc: my bad on the r+. I thought we had agreed on this change. Will check in the future.
- brson: these minor things keep passing through quickly.
- jack: time to talk about changing the process? secondary review for core language features?
- pcwalton: that's what firefox does with super-review
- jack: there is another this morning. 
- brson: another one that expands the escapes for characters and should not go in without additional discussion

## \0 change in the queue
- brson: people hate \x00
- pcwalton: I thought that \0 worked
- tjc: could help for passing constants to the FFI
- acrichto: we don't have a function that lets you safely pass such a string to an FFI anyway. It would be nice to let static strings do this.
- kmc: an abstract named unix socket is created by having the first char of the name equal to NULL
- all: do it

## what can appear in RUST_PATH
- tjc: this is the RUST_PATH hack, allowing it to violate the rule if you pass it. jack wants to be able to put individual directories into the workspace with a special crate file. It's for porting servo over. It isn't necessarily bad, but it complicates the story. cmr also wants rustpkg to find sources in the current directory. Should we add these ad-hoc methods or force people into the one true directory structure?
- brson: what's it now?
- tjc: RUST_PATH may only contain workspaces and will ignore other layouts. Workspaces require an src/ directory. Might be moot because a package id can be a subdir of a current workspace. But lots of people have run into this issue. If I run rustpkg with a lib.rs in that directory, it should just work. Document the behavior, or just make that work?
- felix: could we use a glob at the end of the individual entry in the RUST_PATH to define how the search is to be performed in those directories?
- tjc: that could work
- felix: perhaps over-generalizes and lets people do terrible, terrible things
- tjc: don't use * to keep people from stopping them from abusing it
- jack: if I manually grab a package from github and I'm in that directory, it feels like rustpkg should just work. Also, if I'm working on several libraries that depend on several other libraries, you might hand-craft a RUST_PATH to point at it. Referring to a subdirectory was a big change and really helped. Might not be an issue anymore.
- tjc: let's wait and see
- jack: I don't think it's that confusing to say that RUST_PATH points at the top-level of a workspace or the source
- felix: false matches are the risk

## whitespace in comments 7946
- jack: I put it on the agenda, but it just needs a decision.
- tjc: survey says... inconsistent
- brson: any suggestion on what to do here?
- tjc: no. implicitly, a CR or LF should end it.
- brson: but it's much more complicated...
- kmc: can't we just consider '\r' or '\n' as a single newline token? What's the downside?
- brson: no, but other langauges do even more than that. They have other unicode characters that are line terminators. Is this just comments or elsewhere?
- pcwalton: this is the only place we care. Well, maybe string constants, but the newlines just go into the string.
- brson: does anyone have an opinion? Or an order?
- tjc: that's you, brson
- pcwalton: Go's is both easy and what we already do. How about we stick with that?
- kmc: but the thread/issue started with what we're currently doing...
- pcwalton: I'm not concerned about it
- felix: when do we care about naked \r
- pcwalton: when we use our time travel back to Mac OS 9, which rust doesn't work on
- tjc: so we're worried about a hypothetical editor that munges the files?
- kmc: I'm gonna run some rust on OS 9!
- brson: there is no real use case here. 

## Mavericks
- brson: don't work on OSX 10.9, but we don't work on it. Unwinding does not work
- acrichto: what would this take to fix?
- tjc: somebody has to volunteer to move to 10.9 and fix it, otherwise we move on without it

## crypto
- pcwalton: people we've talked to say do not implement any crypto in rust at all
- brson: including digest functions?
- pcwalton: didn't ask. But we should not implement it in rust. The Go folks have a worse FFI and have Adam Langley who likes to write it and they don't claim it's secure. We're not going to do the work required to write a secure version in all its glory (e.g., timing attacks). Opinions? The downside is that we will have a dependency on openssl/nss.
- brson: can be in the incubator instead of the main repo
- pcwalton: lots of people will have to install/use this
- kmc: high-quality openssl bindings that are memory safe will be a lot of work as well. But a better way forward. Constant-time memory compare we might have to do in-Rust.
- pcwalton: we can try to write it. but can't we just use openssl's?
- kmc: is an openssl dependency too much for just that one feature?
- pcwalton: there's also a backlash against crypto-secure pseudo-random number generators. Use the kernel one only, not the user-space ones.
- kmc: be like go and just use /dev/urandom
- brson: performance is terrible! Graydon looked at this.
- kmc: no, buffer it.
- brson: maybe it wasn't buffered? He definitely looked at it vs. in-user-space and it was quite slow.
- kmc: I get 10MB/sec from commandline
- brson: this affects our hash tables. It needs to be fast...
- kmc: do we care about DOS due to predictable hashes?
- acrichto: slow hash maps will kill us.
- kmc: still need to sync on the buffer
- pcwalton: per-thread buffers
- kmc: let's try it
- brson: consider it, but do not make this change lightly
- felix: do graydon's notes exist on this?
- brson: don't know where they area
