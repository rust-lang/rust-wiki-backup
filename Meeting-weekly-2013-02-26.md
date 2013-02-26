Attending:
* Azita, Brian, Graydon, John, Niko, Patrick, Tim

# Agenda

* Capturing mutable variables (nmatsakis)
* Moving from owned fields and so forth (nmatsakis)
* squash commits before pull requests?
* impl privacy
* forbidding chained imports at top level
* notes on new borrow checker (nmatsakis)
* (something about "single inheritance?")
* `@mut Trait` and so forth (nmatsakis)

# Capturing mutable variables

N: When you make a not-by-ref closure, we forbid you from capturing mutable local variables.

mutable means declared with "let mut", regardless of type

This rule can confuse people. The intent is to prevent mutable vars from getting copied/captured, which could cause confusion. The programmer probably meant to make a mutable box to share b/w the closure and the original stack frame.

People run into this a lot. Might be nice to add the ability to have an upvar that you can mutate, so you can maintain state between invocations of a closure. I thought about saying that you could capture mutable variables, which would remain mutable, but you can only do that if they're subsequently dead inside the parent. We do an analysis like this already in liveness (currently only for warnings).

G: This sounds very much like a warning to me

N: It should be a lint mode with a warning. So maybe I'll just change this to a warning that says "you're copying a mutable variable and also changing it later."

G: Either warn at the capture or at the later use.

N: Seems possible. Feels like a lint mode, yeah.

J: I agree that if a closure captures the value and not the original binding, that's surprising. I support  either a nice strong warning, or the status quo of disallowing it. Allowing a move into the closure also seems reasonable.

N: move into the closure?

J: I understood you to be proposing a semantics where the closure takes control of the mutable binding. Like a move, but not quite like a move.

N: That would be the default, yes, & it would be adjustable via lint. This relates to the option dance b/c sometimes you need to swap mutable options.

# Moving from owned fields

N: Right now you can move from a local variable & you can move from a field owned by a local variable. You can only do that if you write it in a particular way w/ a let.

     struct Foo { f: ~[uint] }
     let v = Foo { f: ~[1, 2, 3] };
     return v.f; // ERROR today
     let Foo { f: f } = v;
     return f; // WORKS today     
     
Semantically, these are the same, so no good reason to give an error for the first one but not for the second. If we permitted the `return v.f`, it would consume `v`. You may want that behavior. Does anyone object?
[agreement]

# Squash commits?

J: 2 diff ways of reviewing pull reqs. You can look at every single commit, or the whole thing with `diff` in github. Should I squash/reorganize so the commits don't look too confusing? I'd like to preserve history...

T: I keep commits the way I want it in the future, not documenting my past thought process

G: Yeah, that's the usual way: the design is you'd clean up a certain amount before submitting a PR, don't 
preserve all steps you did while discovering what to do. Preserve what the reader would want to know.

J: OK, more squashing.

G: Some, but not everything at once, since it makes it too hard to follow. It's a matter of taste.

# Impl privacy

P: Now, putting `pub` on an impl is ill-defined: it doesn't do anything, but it's accepted. People are randomly putting it in. I have two proposals for what it should mean. First: `pub impl for...` has no sensible meaning, so we should forbid all visibility modifiers there. This is b/c you're declaring overloads for *existing* functions, which already have their own visibility.

G: I didn't know this was even a legal form of impls! Is this standing in for default methods?

P: No, that's just how you impl a trait.

G: You said `pub impl`...

P: `for`. `impl trait` for types. I've been using this colloquially to mean the trait one. For the other impl, I think a long time ago we decided this: For just `pub impl`, impl methods on a type, I think the sensible thing is that `pub` makes the methods public by default, `priv` makes them private by default. You can adjust visibility on a per-method basis. If we make this work, people may start using it more. It's like OO languages where you often want methods to be private. You can mark each individual method private or mark the impl private, or have a public impl and a private impl... Everyone OK w/ this?

[agreement]

(Note: this is #3985 )

# Forbidding chained imports at top level

P: Currently you can say ```use io::println;``` at the top level and technically this is a chained import, because the implicit import of the Prelude imports everything from it.


    use core::prelude::*; // inserted implicitly
    use io::println; // comes from the *

This is technically a violation of the new resolve rules. A chained import is an import whose base starts from another import.

G: Yes, sometimes I have to write core:: in front of things and sometimes I don't...

P: There are two ways we can deal with this...

[several]: I like requiring core::io::println.

G: It means you can name io::println if you want in an expression context, but can't use it in an import context.

P: You're right, that asymmetry is confusing

G: Might be enough to motivate removing `use`s from the Prelude -- just leave in types and values

P: That you can't use from it is confusing, yeah

G: Instead we could define our own `println` in the prelude

P: If everyone's on board with that, I'll make this change. It'll probably break lots of code

G: It should go in before 0.6.

# New borrow checker

N: I've been pushing away on revised borrowed checker rules. I was trying to get away w/ not implementing purity. So far, haven't encountered any major obstacles but am being forced to rewrite bits of code that were using interior mutability. Wanted to report on my non-obvious findings.

P: Are you working off newest incoming or have you diverged? We have fully de-muted rustc and syntax now

N: I've missed some of them, but I'll rebase. It'll probably make the work easier. In some cases, I found some of the de-muting was incorrectly done (resorting to unsafety). That's a separate issue, though. It seems like what I see as best practices is that there will be a lot of &mut receivers for traits that you may not think will require mutability. For example, in io, both read and write conceptually mutate their receiver. It's not too surprising, but when you try to make mutability visible, and also try to have abstraction, something has to give (abstraction tries to hide mutability but we want to make it explicit).

G: I've noticed that a few times

N: I don't know if this is bad or not. I've noticed the same in other languages, particularly research languages focused on identifying read-only data. Maybe it's just how it is.

P: I think it's just part of what happens if you put mutability in your type system.

N: A lot of times the answer is "yes, it could be mutable". And you have to be doing that thinking anyway when designing very generic traits.

P: There's a back door, @mut, just in case you need to use mutability where it doesn't belong.

N: I'm happy with that back door for now. I could imagine soneday wanting a different type of back door, which gets back to the question of Cell.

P: I'd like to try to remove Cell; if we have to have it, we can add it back.

N: Just wanted to make sure nobody really hated this direction.

G: It's been a little surprising, in an educational way. Forcing me to be more honest about whether my implementation should be mutable.

# Inheritance

G: Separate question -- P had a throwaway comment about proposing single inheritance and I didn't know what he meant.

P: It turns out Servo doesn't quite need single inheritance, though it would be nice for a few things. Servo will need unsafe code for its DOM implementation anyway, b/c it's managed by the JS GC and that can't be tracked in the Rust type system. (it can't talk about foreign GCs.) That's fine, but it might still come up with layout boxes. You want to be able to create single inheritance trees, and there are a few things you want to have: allocate only as much space as you need for the variant in use (hard to do for enums), first. That's simple. Second, you want a combination of virtual and non-virtual methods. You want shared fields you know are shared between all instances of the trait. An example where this has come up is render boxes: they're in a doubly-linked tree and you want to be able to manipulate the tree structure with the fields at known offsets. You also want virtual methods for things like "is this a block context?" We might be able to make do without virtual methods -- overuse has caused perf problems for Gecko and Webkit. But I have a hard time thinking we'll never need them.

One easy way to make this work is to have traits be able to inherit from a struct, so struct fields would (at runtime) appear in the representation of every instance of the given trait. That's a simple thing we could do. Might also want structs inheriting from other structs. Have to make sure the subtyping relation makes sense, to avoid C++'s slicing problems. I don't know how important this is right now; might be Rust 2.0. We had other ideas (me & N.) but this seems the least intrusive one.
    Example rules in capsule form:


        struct DOMNode { ... }
        struct Element : DOMNode { ... }
        enum Element : DOMNode { ... }
        trait TreeTrait : DOMNode { ... }
        &Element <: &DomNode but Element not <: DomNode

        Can only "extend" from structs with fields, not enums or newtype structs

N: Want to leave options open, don't need to jump on it

P: We don't need it that badly -- I'm simulating inheritance hierarchy with unsafe code (on the other hand it has to be unsafe anyway). If people have other ideas, I'm open to them. Trying to use composition instead of inheritance doesn't work when you have overridable virtual methods and want to make the pointer offsets work. Go just ends up having way more virtual method calls than they need (making everything an interface forces that), as well as virtual field accesses.

N: Or you have to thread around both the upcast pointer and the self pointer.

P: Yeah, and still lots of dynamic checks; it's not pretty.

N: Unless it's dire, I think we should wait till everything's settled. It does seem like it can be added 
without too much intrusion.

# Borrow checker

N: What currently blocks me is that our support for boxed traits is kind of weak. I could make the borrow checker a little unsound.

T: working on it

P: Are we going to have @mut traits? I'd assumed all boxed traits were implicitly mutable

N: I figured we'd do exactly like everything else (@mut means it's mut) I think it's not too hard for me to unblock myself.

G: We still support writing a trait name as a type and treating it as an @trait. How far are we from killing that?

P: I tried once but got confused by astconv. What needs to happen is we need to make traits not use the normal vstore thing. We need to have the concept of a bare trait, only used in type error messages, and a different type to represent where the trait lives. I know how not to do it!

# GC 

G: I revived P's conservative GC over the weekend, going to try to check it in this week. It won't be precise initially.

P: We need to run before we can walk. I mean the other way.
Probably will be some issues with bogus or non-traversable tydescs inside boxes, that we'll need to sort out. 

P: The box annihilator has kept  us somewhat honest WRT the tydescs, but there are probably lingering bugs.

N: It'll be great when copying an @T is cheap

G: If you turn on --gc, there's never been any attempt made to not emit refcounting code, right? And, the existing GC still turns on a bunch of addrspace stuff to do precise stack maps, right? Is there general consensus that this won't work, LLVM won't like it?

P: It's incomplete, I recommend moving it to a -Z flag

B: It's bit-rotted, I think we should remove it completely

G: I don't know how to think about it in the long term. Suppose we find conservative-stack is leaking a lot of stuff, will we just start over from Elliott's notes?

N: We'll probably want to go back in history, read the code, and copy chunks of it for our purposes.

# Libraries

G: Any feedback on the library process proposal? The next step is to talk to the ppl who seem to be doing a lot of library work already, looking at what's gone through bors lately. Ask these ppl if they want to acquire review rights. The idea is letting bors listen to additional reviewers, who can't push directly (but can do so pretty directly by filing a PR and self-approving). The purpose is to make people who just want to do library work feel comfortable taking more of a free hand.
Goal is to avoid having lots of competing incomparable versions of libraries, and to avoid community members competing instead of cooperating. Want to make sure ppl interested in the same topic collaborate.

N: maybe we should reach out to some people on IRC in a private /msg, if we know they're interested. Once the process gets underway, ppl will be more comfortable joining in.

G: I'd like something more formal eventually like in other languages, but we're not there yet. If you have any thoughts, please share them.

B: Right now we don't have much idea of which library topics we want to undergo this process. Personally I want to do IO soon, but that's a vast topic and I don't want it to be first. Do we know what areas we want to address?

G: For a community self-managing process, I'd pick the easiest, most self-contained libs first. May involve some busywork (inviting people to do modest tasks). Lots of libs have bindings to other libs, file formats...

B: Maybe we can start by reviewing some of the more well-trodden parts of core that we're already confident in.

G: Sure! Just doing cleanup topics...

N: Collections is an obvious one, we know strcat's been doing great work, not sure who else

G: I think the things currently written up in the residual list on cargo-central are packages people are interested in, some of which are nearly adoptable at this point.

P: I would like bindings to C libs to be out of core. I think zlib and linenoise shouldn't be in core.

G: There's going to be a lot of those.

P: sdl in std seems odd, though.

N: Are we going to have easy-to-use package management?

G: Yeah, but at the limit, it'll only know how to query local package managers about foreign packages. Figuring out which debian packages to install would be over-reaching.

B: You're talking about native components we don't supply?

G: That's kind of what I'm asking. If we consider std the "libraries we supply", should we also supply the native half if there is one? Currently we do, but could become a maint burden.

B: There are some we won't be able to get away with b/c of debian. 

G: Yeah, debian won't allow that

B:  Particularly on Windows, there's no other option but packaging everything up

G: Yeah, we'll have to do some thinking about how to have optional components. It's not great, but I think this will happen quite a lot

N: Might want to look at the path trodden by the Mac; what Homebrew did. Some limited stuff is available, it's often old and out of date, you often want a new one but don't want too much...

B: Personally, I'd like std to be broken up and managed by rust-pkg, instead of being one big monolith. 

G: This may be a different approach, various IRC people have talked about it: making the pkg manager do more of the work. I want to figure out how to make this a good idea. Std libraries are good b/c documentation; lack of name conflicts; reliability; some degree of community support that's above what you can expect from the ecosystem. Some differentiation between support levels. We can't promise support anyway.
Someone said there's a Go site that tracks all the package documentation and puts it into one big doctree. What I'm reaching for is the notion of differentiating packages. As a project, we should mark some for special treatment e.g. "we don't release if this doesn't work"

B: I agree, I'd expect some are managed & regression-tested by us and we always accept them to be available.
We could have packages still in-tree, but built and installed by rust-pkg

[meeting concluded]
