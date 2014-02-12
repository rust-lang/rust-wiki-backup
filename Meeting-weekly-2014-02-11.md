# Agenda 02/11/2014
* Foo { x, y, z } (acrichto) https://github.com/mozilla/rust/pull/11994
* finally!() and std::finally (acrichto) https://github.com/mozilla/rust/pull/11905
* static values, dtors, and muts oh my (matsakis) https://github.com/mozilla/rust/pull/11979
* removing implicit bounds on procs / traits (nmatsakis) (can't find issue just now)
* crate keyword (brson) https://github.com/mozilla/rust/pull/12017
* install media (brson)
* pizza/food at rust meetup (pnkfelix)

# Attending

dherman, acrichto, azita, brson, larsberg, pcwalton, niko, jack, nrc

# Status

- acrichto: channel rewrite, backtraces, green optimizations, unix sockets, compiler flags
- brson: installation, diagnostics, docs, roadmap

# friend of the tree

- brson: Flavio Percoco (FlaPer87). Contributing since September. Doing issue triage and organizing community events in Italy. Done some optimization in the stdlib, including the `pow` function. Has been really helping with a lot of important but thankless bugs, which I appreciate.

# binding in struct patterns

- acrichto: Right now, you don't need to use the name of the field if you want the variable to have the same name. We wanted to do this for creation so you don't have to duplicate the name (e.g. `for x in Foo {x, y, z}.iter()`. There are some ambiguities in parsing, though, related to for loops, that require an extra comma. What should we do?
```
    for x in Foo { x } { }
```
- pcwalton: I don't like that syntax myself, as it plays fast and loose with lexical bindings and scope
- nmatsakis: Opposed to both pattern binding and expression position?
- pcwalton: Yes.
- dherman: I like it in binding.
- nmatsakis: Feels unhygenic in expression space...
- dherman: I have a paper about why this is a bad syntacitic form, but I still really like it anyway. My argument was that identifiers should have a single role (identifier, non-binding label, etc.). This sugar has it be both a named label and a variable, which breaks alpha-renaming. But, it's such great sugar that it's hard to avoid.
- pcwalton: You can treat it as expanded in the lexer...
- nmatsakis: Yes, that's how it works. It doesn't even appear in the AST. So it isn't really unhygenic, though it feels like it might be. The ambiguity is with the brace... comes up all the time. TBH, it's not the case that all expressions can occur in those limited positions. Maybe it is now? We used to have restricted scopes.
- dherman: You mean a statement `{Foo:bar} = ...`?
- acrichto: No, this is for creating a struct expression `Foo {a}`. If the struct had a single variable, `a` and there was also a local var, it would match here.
- dherman: Oh, that feels bad to turn it into a reference. That said, ES6 is also adding this shorthand. Not that JS is a model of hygenic syntax...
- pnkfelix: is for the only place with ambiguity?
- nmatsakis: if. while. All keywordy brace situations. match, for, etc.
- acrichto: If we have precedent for disallowing certain types of expressions in those areas, we can keep doing that. 
- nmatsakis: Certainly, unusual to write a struct literal there. But it makes the grammar significantly more complicated to formal modeling.
- pnkfelix: I could see wanting to wrap an interator in a struct literal there, though.
- nmatsakis: How many people have a weird feeling about this? Seems like everybody feels bad about it.
- pnkfelix: Thought it was cute but don't care.
- pcwalton: Don't like this change.
- brson: Decided.

# rules on static values

- nmatsakis: We discussed the rules on static values should be w.r.t. linear types or types that need destructors. Setttled on type of a static variable should not be affine/linear. It shouldn't contain anything that needs a destructor. Flavio was trying to implement it, but he found places where we violate it in the codebase. We were thinking about a subtler rule. Can have things like Option<~T>, which is optional linear things that are not there in that constant but are in others:
```
static DEFAULT_VALUE: Option<~int> = None
func(DEFAULT_VALUE)
func(Some(3))
```
- `DUMMY_SPAN`, the static variable in the module ty
- nmatsakis: Came up in a couple of places. The proposed rule was: for static muts nothing changes. for immutable, impose the same restriction, but only on the initializer value. So you just have to have expressions that are all constants and do not require that a destructor runs. So Option<~T> might require one, but as long as it's always None, you don't need to worry. That rule is already enforced by the constant translation code. There is one thing we need to do, which is not zero'ing out moves when they're copying values out of static values (at least until we fix up the move semantics to not zero values anyway)
- pcwalton: Repeating vector literals, too. None,..16 is handy! Basically, can't use the repeating vector literal syntax for anything with a destructor, which means it's impossible to make a fixed-length item all of the times. So you end up having to write out all the constructed values...
- nmatsakis: Could apply the rule there as well. Type must either be POD or the value must fit this restriction?
- pcwalton: Yes.
- pnkfelix: So we'd allow POD with destructors defined?
- nmatsakis: No such thing... maybe there aren't two cases. Was trying to distinguish a reference to a variable on the stack like [x,..5] where x is an integer. That should be legal. But not if x were ~T. Guess it's just the same rule, but not just for static constants but also references to things that are in scope, so long as they do not produce an affine value.
- pnkfelix: That makes sense.
- nmatsakis: Were going to use refinement types, but I don't want to go there right now. The other big use case is interaction with C classes. Often have big structs with lots of things with default values. JS has lots of Nones... but I guess they're just null ptrs, so no problem. But I could imagine it with unsafe variables?
- brson: Workaround today?
- nmatsakis: Static function that returns a new instance instead of a constant. e.g.:
```
static DUMMY_SPAN: Foo = ...;
fn DUMMY_SPAN() -> Foo { ... }
```
- pnkfelix: Except doesn't work in patterns.
- nmatsakis: There are about a half-dozen of these constants in the ty module. There may be more. I asked Flavio not to replace those cases until we discussed this change. 
- brson: Does anybody feel bad about this rule?
- pcwalton: I like it more for vectors than statics! In the case of statics, you can define a function. For vectors, there's no workaround that results in acceptable codegen.
- nmatsakis: Maybe a macro...
- pcwalton: Still bad code. The real workaround is unsafe.
- acrichto: I don't like that statics are different from static muts. Make sense from an implementation standpoint, but feels bad.
- nmatsakis: Yes, it's unfortunately. I'd like to remove static mut - they're evil.
- pnkfelix: Already different due to the use of statics in patterns.
- nmatsakis: You can't guarantee that the value in a static mut doesn't chagne during runtime, which is bad because it may require a destructor later. The one place that makes this feel constistent with Rust is that the mut variants often feel different in the typesystem e.g. & vs. &mut. Not exactly analogous, but close.
- brson: Let us please do it. Niko: make it be done!
- pnkfelix: Should we have Flavio wait?
- nmatsakis: He was willing to work on this.

# crate keyword

- brson: Changed extern mod to extern crate (decision). But making crate a keyword raised some issues (in 12017).
- acrichto: Why did we do that change again?
- pcwalton: Because it's not a module; it's a crate.
- pnkfelix: At least one person has claimed it is a module... but I can neither confirm nor deny that position.
- brson: extern mod looks really ugly.
- dherman: How about a contextual keyword? So it's just an identifier?
- brson: I don't like them because we don't do them right now.
- pcwalton: I'm fine with it.
- nrc: Just call the whole thing (with space) OK?
- dherman: Can we use tab then? :-)
- nrc: Ok, contextual keyword seems more reasonable.
- acrichto: If we do go down that route, do we do it elsewhere, too? 
- nmatsakis: What is the problem? Do people think crate is a useful word?
- brson: Mainly because the patch takes away the word crate, which makes a huge change for the Rust compiler.
- nmatsakis: Common compiler problem, since it talks about the things that implement the keyword... I think we should just make it a keyword, make the change, and move on with it.
- brson: This is also Flavio! He replaced crate with crt, which I would not suggest. Maybe crate_?
- nmatsakis: There's precedence for that!
- pcwalton: strcat did point out that if you want to link to C libraries, you might not want to say extern crate for C libraries. There was some proposal for extern mod...
- acrichto: This was a Future Rust where you can pull in a header file and it would automatically generate the types, etc.
- pcwalton: Maybe use `extern C mod` for that and `extern crate` for Rust libraries, because they're completely different.
- dherman: If you have `extern "C" mod` vs. `extern "crate" mod`... ugh, too verbose.
- pcwalton: I'm in favor of the contextual keywords here.
- nmatsakis: The only argument against them is complexity in the lexer/parser
- dherman: it would be nice to have contextual ones for `in` and `out` so that you can use them as variable names.
- nmatsakis: I think it's weird to have crate be the only contextual keyword. 
- brson: You can always make more things contextual in the future; it's back-compat.
- dherman: Maybe, step 1, reserve crate. Step 2, should we un-reserve both crate and in? Then it's two separate questions.
- nmatsakis: I guess contextual keywords are pretty much inevitable.
- pcwalton: crate is not a big deal. box had horrible fallout in Servo.
- brson: OK. crate is a full keyword. And suggest a contextual keyword later. And crate is crate_ instead of crt.
- pcwalton: maybe rust_crate?
- brson: I don't like it because it's a long name.
- pcwalton: the_crate. krate? I like that.
- nmatsakis: I like it too, but thought everybody would shoot it down.
- pcwalton: maybe in servo: box_ to boks? bawks?

# Pizza

- pnkfelix: Pizza. In SF, have you had food provided? Does the team pay?
- brson: Eventually.
- pnkfelix: So, in paris, can I do that?
- azita: Yes. It automatically comes to the department.
- brson: Here, we file a ServiceNow ticket and it happens.
- pnkfelix: Wanted to check because a game meetup happened and paris did not reimburse.
- azita: Probably the same way. Just file via ServiceNow, talk to me if it doesn't work.

# finally macro

- acrichto: I think niko's try...finally change nullifies it. Did you leave the finally method in there?
- nmatsakis: Yes, though I dislike it. It's there for back-compat for the simple case, though I don't know how often that arises. Does everyone know what happened there?
- brson: yes.
- acrichto: There are times when you want to run thing on a destructor... a lock is a good one. Every good lock will have a RAII guard that handles it. Stick with the closure syntax, or use a macro? Like deferred syntax in Go.
- nmatsakis: I don't think this is remotely sound, unless it doesn't close over any borrowed data. That's why try finally was set up the way it was - don't give the user a value they can move around. You get something on the stack, right?
- acrichto: Yes, finally! puts a thing which runs a closure when it goes out of scope.
- nmatsakis: As long as you gensym a symbol, you should be ok. it's just a problem if you can do `let x = finally!` and then can move x. Or if you can mutate the contents, as you could set up cycles, observe uninitialized values, etc. 
- acrichto: Right now, we have the finally syntax which is nice because you put it after the code itself, which flows nicely. Now it's up above. The other bad part about the current syntax is that the code you run is in a closure, so you can't move into it, which is only true for the finally code.
- nmatsakis: I find the current quasi-method we use today to be a failed attempt to look like other languages. I'd rather just have the macro-based approach and closures, but I don't care that much in the end. Be curious to know how often it comes up that you don't need to share mutable state. 
- acrichto: A bunch are in the extra::sync module. But I don't know a lot about those.
- nmatsakis: Looking to see, after my change, to see how many remain. There area  few, but mainly in that module.
- pnkfelix: Would you be placated if the name of the macro were changed and the word finally appeared in the middle instead?
- acrichto: Were going to try to use a try macro...
- pnkfelix: Try might be taken. And some people don't like it.
- nmatsakis: Again would be nice to have more flexible macro syntax.
- brson: No doing because it's unsound?
- acrichto: Niko was pointing out that if the macro expanded into a variable you could move, then you'd be in trouble.
- nmatsakis: alternative:
```
let _ = &TheThingWithTheDestructor(|| ...)
```
- brson: So not against it?
- acrichto: Not because of soundness.
- brson: What are we talking about now? Syntax? Semantics? Naming?
- pcwalton: I like defer. It embraces RAII. Defer feels more like it embraces RAII instead of more like try...finally. 
- dherman: Main difference between RAII and try/finally it's that RAII is inline in statement form and continues implicitly vs. try/finally uses nesting?
- pcwalton: Also, RAII unifies finally & destructors.
- nmatsakis: I prefer defer as well. That it doesn't appear after the code doesn't bother me. It lets you put your setup and cleanup code together as well, which I like. All things being equal, I prefer that.
- pcwalton: Without once-fns, the body of the try block is less convenient to use in a try/finally setup with two closures.
- nmatsakis: In the one I wrote, you can pass in data to the body as well.
- acrichto: So, we're mostly in favor of defer...
- pnkfelix: Wait - some people have said that the defer keyword in Go is based on the whole function exiting instead of the block
- nmatsakis: Defer in Go is not what we want to do.
- pcwalton: Not suggesting exactly as defer works in Go. The semantics of defer in Go are quite impressively complicated.
- pnkfelix: I think people just didn't want us to commit to defer semantics in Go or to confuse people.
- acrichto: So we will have this defer/finally thing. The proposed name was finally!. Do we want defer!
- pnkfelix: How about on_drop!
- dherman: I find compound name bad.
- pcwalton: I am OK with using defer!
- dherman: Allow me to quote myself, "just because your semantics is not identical to somebody else's doesn't mean you shouldn't use common terminology, since most people don't know the corner-case differences"
- nmatsakis: does that mean we should use finally?
- dherman: I think the idea of deferring to the end of the block makes sense, whereas finally is tied to try/finally. 
- brson: I agree defer feels good.
- acrichto: Do we want this to be a macro or a function that takes a block? The function needs two bars...
- pcwalton: The function will borrow things for too long. Probably needs to be a macro to do the unsafe borrows, etc.
- nmatsakis: How would it look as a function?
- acrichto: `let _ = defer...`... nevermind. Let's do a macro.

# Implicit Trait Bounds

- nmatsakis: couple of places where we do implicit bounds on things. Example:
```
~Trait => ~Trait:Send
```
- nmatsakis: I don't like this. It's non-compositional. Type Trait works differently under a ~ or an &. Did it that way because it's usually what you want. And requires you have to do `~Trait:`, which is non-obvious. Once we can separate traits from the ~ (in DST), we will want this.
- pcwalton: Want to try this. We did `~Flow:` to get rid of `~Flow:Send`, and it was never obvious when or why you had to do this.
- acrichto: Almost all trait objects are send in io, but not freeze. It would be great if we had precise error messages. e.g. instead of jsut "not Send" but "not Send because of fields...". Then compiler could say "might want trait:Send".
- nmatsakis: Agree that would be great. A little orthogonal, but great.
- larsberg: agree with acrichto; we run into that in Servo sometimes with very deeply-nested structs and playing "find the non-Send"
- nmatsakis: I will open an issue on it.
