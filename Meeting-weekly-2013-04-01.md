## Agenda

* prefixing labels with '
* impl PATH [for …] { … }
* fast-ffi
* plans for copy/Clone
* 0.6 prelease build

# attending

azita, pcwalton, brson, dherman, graydon, jclements, jack, tjc, nmatsakis

# 0.6

* brson: put together a build; close to tip of incoming. a few significant issues came up. one person reported an OS X error. there's an issue extracting the tar on windows.
* pcwalton: the repl is still broken for a lot of people. from leaks on exit to segfaults.
* brson: a lot of discussion on reddit
* jack: i've been building fine on OS X.
* brson: i've also been building fine on OS X.
* graydon: is this the OS variable?
* brson: yes.
* graydon: i'll email them. he replied saying it was solved. the tar extraction was an issue in 0.5 as well. as far as i know it doesn't break the build.
* brson: not a huge problem. we've been telling people to use the installer. there's an issue with binaries getting set to have executable stacks, which is a problem on SElinux. may indicate that the build doesn't work on fedora.
* graydon: that's worth looking into
* brson: strcat gave us a fix for it, but we haven't confirmed that it works yet.
* brson: https://github.com/mozilla/rust/pull/5647
* graydon: we are supposed to have a buildbot to check that. did anyone else run builds to see if they work? please do.

# prefixing labels with '

* pcwalton: niko brought this up a while ago. that would make labels work as lifetimes. there's a question if we want to break labeled continue syntax. ticks make it less ambiguous. do people like this idea?
* graydon: thumbs up
* brson: do we have an example?
* pcwalton: like this:

```
    loop foo: {
        hello();
        break foo;
    }
    
    'foo: loop {
        hello();
        break 'foo;
    }
```

* pcwalton: This lets you use 'foo as a lifetime to mention the lifetime of that block
* tjc: i don't think peeople know about labeled break and continue, so if we want to break it, now's the time. it's nice to make it look different from an identifier.
* brson: does the syntax work in other positions?
* pcwalton: this opens the door to that, but the old label syntax wouldn't work for other things.
* brson: sounds good to me.
* nmatsakis: i'm all for it

# impl PATH for TYPE

* pcwalton: the thing that goes after `impl` has to be a type that belongs to this crate. today it is `impl TYPE for TYPE`. it's a little strange because we have to reinterpret the type as a trait reference. there's a grammar production that encompases nominal types and traits: PATH. it would simplify the grammar to use `impl PATH for TYPE`. this may be just a grammar change with no user visible issues. what do people think?
* brson: it's ok

# fast ffi

* pcwalton: i have an LLVM/Rust patch called fast-ffi. the performance numbers are 15% faster in trans. it doesn't seem to have a large impact on other passes. strcat could come up with microbenchmarks where this helps a lot. what it does: instead of doing a stack switch, there is a new LLVM function level attribute called stack segment which causes stack seg to become large. i'm surrounding that with a wrapper for a c function that contains that attribute. added an attribute called fixed-stack-segment for Rust functions. rustc avoids the stack growth and avoids jumping back and forth for stack segments. design is based on GCC's planned implementation for split stacks. it's called more-stack-no-split in GCC. there's a couple of issues. because the last stack segment is cached, you can end up with any task that called C can hang onto a large stack segment. my plan was not to cache large stack segments.
* brson: it's worse because you'd have to reallocate them
* pcwalton: that's ok. that's how it works today
* brson: it's not. there is a per thread and a per task cache
* pcwalton: i've not implemented that, but it's neeed for scalability. there's some inlining stuff that also needs to happen. performance numbers are encouraging. how do people think about this?
* brson: i have one concern. teh last design we came around to had the ability to specify stack size flexibily. this has fixed stack segments.
* pcwalton: you can use the rust stack
* brson: i don't like hte rust stack approach because it's in the red zone. it's not right.
* pcwalton: the earlier approach has a branch. seems like a waste
* brson: `<didn't catch>`
* pcwalton: you could add an intrinsic, but ugly. there's a better approach. we could add one llvm_used to mark a function as having ai fixed stack segment. it would be a toplevel intrinsic call where you call a function llvm-fixed-stack-segment and give it a function. might play havoc with inlining.
* graydon; let's discuss this on IRC in a more problem solving fashion.

# plans for copy/Clone

* pcwalton: goal is to get rid of copy keyword and move it to clone trait. 
`impl<T:Copy> Clone for T { … }`
this impl is not coherent. the problem is that you can't know whether it's copyable because it could have generic type parameters and you don't know whether those are copyable. if it says they are not copyable, it's wrong because you can have copyable generic params. i would like to say that the clone trait is auto-derived for any nominal type that is implicity copyable and has no type parameters. coherence synthesizes defids.
* graydon: why is the `T:Copy` bound not enough to indicate implements-copy?
* pcwalton: it is enough. it's hard for coherence to make that decision. coherence has to check to see if trait implementations conflict, and it's hard to do with generics. example:

```
    struct MyStruct<T> {
        x: T
    }
    
    impl<T:Copy> Clone for T { … }
    impl<T> Clone for MyStruct<T> { … }
```

coherence detects this as a conflict. the question is do they conflict? the answer is possibly, depending on what T is. if T is copyable, then yes they conflict. but if it's `MyStract<fd>` then they do not conflict.
* tjc: it's about more than just copy
* pcwalton: our answer to overlapping instances is to disallow them.
* tjc: the exmaple could occur for other traits than copy.
* pcwalton: right now we disallow and we have to because there is some T for which there could be  conflict. i worry that if we allow this we'll have problems down the road. if disallowed, you can't put a destructor on a generic struct or enum. it would rule out `ARC<T>`.
* tjc: what makes this extra difficult is implementations would span crates. one solution from research is to write impls as if-then-else. ti's harder to do this cross-crate.
* graydon: ... why can't you write `impl<T:Clone> Clone for MyStruct<T> { ... }` ?
* pcwalton: this is also conflicting because tehre are some types for which both copy and clone apply. for example int.
* graydon: ok, I don't even know what the difference between Copy and Clone is here
* pcwalton: copy mean implicitly copyable. clone is just a special trait that can be overrideen that the compiler doesn't know about. in my proposal it will auto-derive but wouldn't invoke it for you. the methods that take `T:Copy` would take `T:Clone` instead so you could do ARC boxes and bump the ref count, etc.
* graydon: I was asking why you can't put the bound-you-need on the `impl<T>` part
* pcwalton: because there is no bound. You need `T:!Copy`.

   `<T:!Copy>`

we don't have a function with a bound that doesn't implement a trait. i feel like it would be a lot of complexity.
* tjc: the negation thing is also in that paper i was referring to (instance chains in haskell). it's recent technology. would it be easy to solve the problem with trait negation?
* pcwalton: i'm not sure it would be easy. part of the issue is that whether it implements copy depends on whether it implements drop. in my proposal you don't need Copy at all. you just have Clone. it's a lang item and the compiler derives the obvious implementation (type implicitly copyable).
* tjc: would it be ok to have an impl for Clone if it has an impl of Drop?
* pcwalton: yes, and arc already does that. it seems simpler than having trait negation.
* nmatsakis: what would you do for generic structs
* pcwalton: you'd have to write `deriving(Clone)` on those
* nmatsakis: if write `let a = b` and don't call clone, it has a certain semantics.
* pcwalton: when the compiler auto-derives is tied up in semantics of implicitly copyable.
* graydon: I'm ok with this. it feels clunky but so does everything in this space.
* pcwalton: instance chains would allow us to reify this automatic deriving and notion of implicit copyability in a generic way
* nmatsakis: the main argument for auto-dervie is because people forget to do it
* pcwalton: you want one off structs to be easy to use.
* nmatsakis: one off structs already need Eq. this is only necessary when using them generic. i'm not opposed to making them automatic.
* pcwalton: sounds like mostly have consensus.
* nmatsakis: we can discuss precise details later

# rust release

* tjc: are we de-milestoning the rest of the bugs?
* pcwalton: a lot of them are my fault. i'll try to go through and resolve them.
* nmatsakis: has anyone updated regions tutorial? if not i'll do it on the plane.
* brson: do we need it updated by the time we tag the release?
* nmatsakis: it may have some gross inaccuracies. do we plan to tag tomorrow?
* graydon: I expect we won't tag before tomorrow
* brson: now is a good time to slip in any documentation that's also missing
* graydon: I'll do another run over the remaining bugs
* nmatsakis: IMO, unless the fix is ready, it gets bumped.
