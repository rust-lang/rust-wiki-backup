# Agenda 10/01/2013
- ~fn changes (brson) https://gist.github.com/brson/6781740
- Submodules in rustpkg, as per https://github.com/mozilla/rust/issues/9514 (tjc)
- Semantics of `rustpkg test`, as per https://github.com/mozilla/rust/issues/9003 (tjc)
- removing float (brson) https://github.com/mozilla/rust/pull/9519
- clone and deep_clone in prelude (brson) https://github.com/mozilla/rust/pull/9521
- `rust` tool maintenance (tjc)
- struct fields private by default (acrichto)
- privacy https://github.com/mozilla/rust/issues/8215 (acrichto)
- hoedown (acrichto)
- composition in option (Cell vs Mut as well, https://github.com/mozilla/rust/pull/9359) (acrichto)
- drop in static items (acrichto)
- fixing on osx 10.9 https://github.com/mozilla/rust/issues/6849 (acrichto)
- raw string syntax https://github.com/mozilla/rust/issues/9411 (pnkfelix)

## submodules in rustpkg
- tjc: local changes to a package that is in git. Right now, we overwrite those changes. 
extern mod foo = "github.com/mozilla/whatever";
(and then check out sources for whatever into src/whatever)
- tjc: not ideal to have to distribute the source code for the repo with your distribution. Instead, we can put the fetched sources under build. rustpkg will look in src/ first, though, so you can have a local version. 
- tjc: build/ is the one rustpkg fetches and makes readonly. If you want to make local changes and use them, check out a copy under the src/ directory. Then rustpkg will notice it first and ignore the one under build/.
- brson: manual process to put it in src?
- tjc: one way might be manual. The other is to just copy the version from build/ into src/. The other other way is to put an attribute on the ```extern mod``` that you want to edit it, and then rustpkg could create a repo for you under your src directory.
- acrichto: you'd want to be able to commit your changes
- tjc: you don't necessarily want to be committing those changes constantly
- acrichto: it would be unfortunate if people accidentally break themselves by having a tweaked version in src/ and then committing dependencies on that 
- tjc: hopefully this will be clear enough to avoid it
- pnkfelix: fine by me. enforce the read-only-ness in the build/ directory? Should we error if they touch it?
- tjc: not my initial plan. But maybe it'd be less error-prone if we did enforce readonly.
- pnkfelix: either make it readonly or if you have magic to look around, make sure that build/ is not writeable at the same time that it finds the src/ directory (since those changes in build/ are being ignored).
- brson: if I want to check out everything in servo to edit it, what's the process?
- tjc: two possibilities. An attribute that rustpkg knows about so that it will put all the sources in the src/ directory. Or we could not do that and you just move things around manually. There are 12+ modules, so that might be a lot of work.
- brson: two modes: Servo builder and servo developer. Any source code change that has to affect that mode doesn't seem like the right place to do that. Developers will probably want to say, "give it all to me, editable."
- tjc: maybe that's a good flag?
- brson: used to have a configuration file that had a bunch of crate attributes that get merged into your crate. In any case, it needs to be more automated.
- tjc: not a manual process, nor should it require editing sources.
- brson: how does go do this?
- tjc: don't know
- acrichto: Ruby has that with its gems. But that is still manual and breaks for other people.
- tjc: is that a command?
- acrichto: it's a gem file that contains all your dependencies. You run that to set everything up.
- tjc: trying to avoid having something like a gem file for Rust, if only to avoid having more files that might get out of date. I'll do more research on Go/Ruby.
- dherman: hit up steveklabnik, who's also in the Ruby community and should have some ideas.
- tjc: have before. He said, "if you have a gem file...". Will follow up again to try and understand the space and scenarios a bit better.

## clone and deep_clone in prelude
- brson: mainly for passing stuff to higher-order functions like ```map```. Since it's in the prelude, we need to talk about it.
- dherman: what is deep_clone?
- pnkfelix: is this a sort of code smell? It's associated with an ugly pattern, but it's still not a good pattern.
- tjc: what happened to the function syntax change proposal?
- acrichto: I don't think it worked
- pnkfelix: should there be a different iterator that delays cloning?
- nmatsakis: should work with the universal method call syntax
- brson: clone::clone, then. We also decided the universal syntax wasn't so important we would do it
- pnkfelix: is map of clone really what we want people to write here?
- acrichto: not a fan of making it easy to do this.
- nmatsakis: you want a by-value iterator?
- acrichto: we have it, but you need to have ownership of the vector itself. could have a clone_iter that clones everything and gives it to you by value...
- tjc: it's easier for the person learning to write it without using the special iterator.
- nmatsakis: map of clone doesn't seem particularly intuitive
- pnkfelix: I was thinking having the alternative iter would be just as easy to find
- nmatsakis: clone as a standalone function might be useful in the context of auto-deref.
- brson: how many standalone functions are we going to add to the prelude? from_str is a static method, and calling from_str::from_str is ugly, so we put it in the prelude, and that's true of many things
- acrichto: similar with stat. all of the standalone functions is an unfortunate pattern.
- brson: the universal syntax and a way to import both static functions as well would make this nicer
- acrichto: generally against this proposal
- brson: resist for now
- nmatsakis: is it just the prelude part?
- pnkfelix: put it somewhere else?
- nmatsakis: could just put the clone method on clone and not put it in the prelude, but that's a bit duplicate as well
- acrichto: importing would be nice, but importing would be pretty ugly...
- pnkfelix: the standalone function would at least make that cleaner
- acrichto: less resistance to the change without the prelude addition

## composition in option
- acrichto: we have get, get_ref, get_mut_ref, etc. tripling the size of Option. Instead, we could use as instead
- acrichto: the PR is to collapse several of the types together. But it's another big change to an API that's already in flux.
- tjc: The PR is pretty clean, but you might have to write more code to use options.
- acrichto: if you do not want to consume ownership, then yes it's more code
- nmatsakis: not a fan of the names, because they don't tell you much if you don't already know the names. I prefer `as_ref` and `as_mut_ref`
- nmatsakis: bug about how the borrow checker could be more lenient than it should be. I'm afraid that this code may be triggering that bug. The pattern should typecheck regardless, though some instances might have to be corrected.
- brson: agree that they're better. Maybe we should make all of those changes now as part of this.
- acrichto: didn't we previously have `get`? the change would be less words.
- brson: 
- nmatsakis: Jack suggested that unwrap suggests that it moves even when it doesn't. Part of the problem there was just the name unwrap, so get might be better
- acrichto: Just looking for other unwrap functions, we have them on Arc as well. If we rename `unwrap` to `get` that would be going back.
- brson: let's address it in a separate issue.
- acrichto: it's unfortunate that the pattern explodes into something large.
- tjc: if we make options too hard to use, people will get turned off from them, and they're pretty core to Rust.
- nmatsakis: we could still have get_ref. We don't have to remove all convenience forms.
- acrichto: most of the changes are of the form map_move to map, etc.
- acrichto: for now, because get_ref is so common, we try leaving that. So we accept the PR with changes (leaving get_ref) and otherwise take it in its entirety.
- dherman: We don't have a global naming convention yet around ref, move, and copy yet. Because they're so central to the language and consistency is so important in the standard library, it'd be nice to get there.
- nmatsakis: that's what this PR is trying to close down.
- brson: We have had a bunch of PRs brining everything into line. We do need to document it.
- brson: prefer as_ref and as_mut_ref?
- pnkfelix: as_mut OK for as_mut_ref?
- acrichto: as_mut does not imply that it's a reference.
- brson: all the existing ones... no, some say mut_ref and some just say mut. get_mut_ref vs. map_mut.
- dherman: If you're getting it as mut, could it be anything other than a mut ref? Seems weird to get a mut box.
- brson: would as_mut extend to other types? Or do we need to be explicit about refness in other containers
- pnkfelix: either @mut or &mut
- dherman: On naming conventions. If you have a common refectoring pattern, if it could have the same number of columns, you avoid rippling the change across the codebase. FWIW.
- brson: as_ref and as_mut. Let's open another issue for that.
- dherman: unwrap is the mut one? I like the image: it's like candy. When you unwrap it you throw out the wrapper. But it doesn't extend to other APIs well.

## raw strings
- pnkfelix: Lots of opinions, but no clear leader. Either a string syntax that allows for few escape sequences and handles the common ones nicely. e.g. double quotes to delimit the strings and only need to escape double quotes with double-double quotes. I'm not a fan of it, but it's a good example.
- brson: Why not that one?
- pnkfelix: A use for me of raw strings is to emit code from other languages, so there will be lots of double quotes in those other languages. My bigger complaint is that it should really be raw: copying it should be identical to what will be printed. The only way to do that is via customizable delimiters, which might be based on the content. C++11 has this (as do other languages). 
- dherman: heredocs
- pnkfelix: If we do that, we can either adopt somebody else's syntax (Kevin documented the possibilities well here), which I can see. Otherwise, we could come up with a special syntax for Rust. I was originally a fan of the C++11 syntax. The variants specific to Rust:
  Kimundi
```    r" adksafjasaifj raw text   "
    r#####"  afkafsksakaksfasfa " laksjalklkjfaf " alkfjkla #"    "#####```
 Felix's original idea
 ``` r" afklflasflkafkl raw "
   r#some-char-sequence#"  fafajfsjsafkjsafjkaf  "#some-char-sequence#```
 Dave's Mirror Image note (potentially from Fortress): e.g.:
 ```  r#(<#"  aklffaklfslkjalj lk  alkjs  "#>)#```
- pnkfelix: the idea is to precede the quote with hashmarks and match the number of hash marks before and after the quote. 
- dherman: Other precedents. Perl quotelike, but there's also this concept in regular expressions and it's in Fortress as well. People can create arbitrary mirror-image tokens. A left brace would be right brace, etc. e.g. -=} matches {=-
- pnkfelix: Lua's approach is like that, using brackes to break things up. The thing that scared me about the syntax is searching for that line of hash marks. The alphabet of a single character sort of limits your choice. Was considering something more like defining the char sequence.
- dherman: So that seems like it might be too general?
- pnkfelix: I thought it would be easier for people to scan, but people didn't like it. The nice thing about Kimundi's approach is that it's compatible with Felix's idea if we decide to do it later.
- dherman: In agreement: roc made a blog post several years ago that the more syntactic variation you offer increases the ability to have a style war. Cut it off if possible. The hashes are just expressive enough that if you have three levels of nested code quoting, 3 outside, 2 inside and 1 inside that should work, modulo corner cases. In practice, the scanning isn't for N hashes, it's just for a pile of hashes after a quote. The human reader won't really need to care about it.
- pnkfelix: either a lightweight delimiter syntax or one that's growable, but could become ugly. We don't have to commit to one right now, but do we have a preference as to style? Or should we just punt raw strings for 1.0? But I think we really want them for format.
- acrichto: The initial impetus was format strings and escaping. But regular expressions will also be a big deal. So I'd try to just make it easy to use so it doesn't look too odd. I'm kind of a fan of the ```r"..."```
- pnkfelix: We could use that as a baseline, if we want.
- dherman: We should have very concise defaults. r" matches that.
- brson: Sounds like consensus. Should we also add the r# ?
- acrichto: We need some form. I was doing giant blobs of HTML quoting and it would help.
- pnkfelix: I haven't tried the hashes in the lexer to make sure it works. We could use something else, if people want.
- acrichto: maybe r{snowman}? (r☃ this is some text ☃)
- dherman: then you have to allow Unicode poo pile.
- nmatsakis: let's stick with hash marks, then.
- brson: pnkfelix, can you follow up?

## removing float
- pcwalton: Let's remove it. 128-bit floats are not happening. The argument is that there may be a 128-bit floating point in the future. As a graphics person, I don't see how we could get there. In C, there's no generic version - you always specify the size of the float.
- pcwalton: People who write that kind of code care about the size.
- brson: we can always add f128 if that happens.
- pcwalton: then apps won't seamlessly migrate. But I don't want them to.
- dherman: We do that for ints?
- pcwalton: that's what C does. ILP64 instead of LLP64.
- brson: everybody's not that fond of float?
- tjc: if there's no use case, then why have it?
- dherman: have to remove the keyword?
- pcwalton: not keyword, just pervasive
- dherman: int, too?
- pcwalton: yes. But not true or false, despite my objections...
- brson: Gone. This seems somewhat casual...
- pnkfelix: should we reserve it to stop people from using it?
- dherman: I think he's joking
- pcwalton: we can add new primitive types in a back-compat way. They'll just be shadowed by folks who have made their own.
- dherman: Is it that smooth for future-proofing to add new pervasives? In Javascript it's miserable. Are you saying we can add them at any point and we will never break code?
- pcwalton: import * may not be true and possibly messes up the ability to add anything to the library. But it's just a feature that we reserve the right to break with our updates.
- dherman: We should try to message it clearly. We will probably make people upset. It's a dangerous feature from a code perspective.
- pcwalton: Even worse! It imports private methods as well (which it has to, because they're not determined by the time we do resolution). You can't use them, but if you reference them you get an error. 
- dherman: Scope is so important to get right for type soundness. I'm not arguing to change how that works, but I'd like to understand it better.
