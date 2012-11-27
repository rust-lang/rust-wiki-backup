# Agenda

- Build systems
- Imports, exports

# Build systems

... discussion ensued ...
- G: Removing crate language puts us one step slightly further down the path of "hard to analyze what is going to happen when you run a build", declaratively
- G: we were already a ways down that path though...
- G: it would be nice to be able to emit a cached description of all the dependencies of a particular run. Unfortunately some of the dependencies are environment-dependent (or at least "configuration depenendent")
- G: Note this is same sort of task we'll need for integrating with IDEs (xcode or MS build project files). These systems all have the notion of specific named configurations. not just random has-foo / not-has-foo configuration probes. The use of named configurations means they can capture >1 build config while still producing a declarative dependency tree within each.
- B: can we capture the dependencies dynamically
- P: syscalls can be tricky
- G: redo / fbuild etc does not use syscalls
- G: rather you have these declared dependencies, which are the initial set...
- G: but when you run the rule, it will record its dependencies for next time
- G: and store them in a database
- G: does require that you be able to capture dependencies while you run
- G: but this is usually an option
- G: fbuild actually hooks in at the Python layer and instruments calls
- G: which we presumably will not do

# Modules have an empty scope

- P: I've been talking about our module system with Dave
- P: Interestingly our system can be viewed as a clone of the ECMAScript modules
- P: Surprise it has all of the same complications---so powerful that resolving names is pretty hard
- P: There are a few issues
- P: One issue is that my scope includes not only the current module
- P: but all the containing modules, making it difficult to know what is in scope
- B: is this a problem for the algorithm, or just humans?
- P: it can be a problem for the algorithm because it leads to cyclic dependencies
- P: in effect you are importing * from all your parents
- B: I have noticed that importing * from a child creates problems
- P: we might be able to handle these cycles
- P: but it's unclear that this is desirable in the first place
- B: what about crate imports?  right now we import core::* at the core of every 
- B: what about importing other extern mods?  do you really want to do that in every file?
- B: think "extern mod std;", do we have to right that in every file?
- P: that's totally fine because your imports, by default, go from the root scope
- P: barring inter-dependent imports, all imports will resolve crate relative
- P: so writing "extern mod std" at your crate and then "use std::foo" in each module
- P: will work out fine
- B: you'll have to add 'use std' everywhere
- N: do you have to?  if you wrote e.g. std::list it will resolve ok
- G: I am confused because doesn't crate relative paths imply you can refer to the root?
- P: right I think you have to say that the parent items are in scope but the modules are not
- P: I think you have to say that the module tree in scope
- N: I don't understand what that means
- T: how is that different from now?
- P: now all items are in scope
- B: I like the idea of requiring absolute paths
- P: there are coherent semantics either way
- P: you can say that only the top is in scope or all parents are in scope
- B: that allows you to have a model where you are either not using anything outside your file
- B: or you must include every path
- N: would you want to add some sort of relative syntax? or just spell everything out globally
- B: I would rather not have a relative path syntax
- P: so, chained imports are useful 
- P: which brings us to another issue
- P: currently the way those are processed you have to process the imports in order (mostly)
- P: so `use foo::bar; use bar::baz;` is ok but the reverse order doesn't
- P: but this is the only place that order is significant
- P: Dave is not a fan and I think it's unfortunate
- P: we could get around it by toplogically sorting the imports
- P: it is kind of weird because of multiple namespaces (type, value) 
- P: we'd have to have to some sort of restriction whereby if you import 
- N: if we say that you always get both namespaces at once or does that not work / prevent useful things
- P: (ed - an example of something tricky that I didn't fully catch)
    mod a {
        use b::a;
        use a::...;
    }
    mod b {
        fn a() { ... }
    }
- P: here the `a` that comes from `use b::a` is a function
- G: can we go back a sec
- G: this issue has come up before and I carry scars from earlier versions of ES
- G: the last time we talked about we settled on the idea of `use mod`
- P: the problem is that if we merge type/module namespace it doesn't work
- P: in order to be semantically coherent we'd need require `use type`
- G: sure but suppose it imports both types and modules
- P: well then what does use without mod import?  does it not important types?
- G: I don't know
- P: the idea before was that `use x` would not import a module but that doesn't work
- P: if the module/type namespaces are merged
- P: so I think that `use mod` is not the best approach
- P: so basically we try to do what we think the user intended and it might not quite be right
- P: (enters an example:)
    mod a {
        fn d() {}
    }
    mod c {
        fn a() {}
    }
    mod b {
        use c::a;
        use a::d; //~ ERROR
        ...
    }
- P: here you will get an error because it effectively expands `a::d` to `c::a::d`
- N: so effectively there is a pre-pass that is able to expand everything to absolute paths
- P: exactly that's the topological sort, effectively
- P: oh, and another restriction, lifted straight from samth's proposal for ECMA Script,
- P: if you import *, you cannot import from modules that were brought in by the *
- N: we could just say that use is absolute, full-stop
- B: I like that, it's more understandable and only slightly more onerous
- G: one additional possibility: add `.` and `..`
- G: that's the only reason you want relative imports almost ever
- G: that makes things relocatable which is a nice feature
- P: mostly a question of syntax, doesn't cause problems
- P: ..::foo seems like a non-starter
- B: that seems much simpler
- G: I'm into it if we can find a way to spell it, so we should all bring up emacs!
- N: I feel like chained imports is not that important given the braces
- P: only language I know where chained imports is common in Python
- G: so when I say you want to import your current module, the purpose would be to chain, right?
- P: that doesn't have to be chained imports but might rather be for importing from children
- G: if I can refer to `.::` is it any different than what I can do today?
- P: it's different because it doesn't search through the entire scope hierarchy
- G: but can you still setup a cycle?
- P: with * and pub use, you can certainly create cycles
- P: several issues to sort out: 
    - ergonomics (making sure you know what's in scope and why)
- G: accidental collision, order dependence, accidental cycles
- G: are those the 3 problems we're fighting here?
- P: yeah I think that's it
- P: other issues too, multiple namespaces make things harder
- G: can you be more specific about precisely which problems they are causing?
- P: yeah it leads to the problem I wrote up above with chained imports
- G: so if I understand correctly adding a restriction such as
    - "all paths have to be absolute"
    - "but you can say . and .."
- G: improves at least the 1st 3 problems but doesn't enable chained imports
- P: I think that with chained imports there are three options
    - forbid them
    - accept some restrictions (topological sort)
    - sacrifice order dependence
- B: it seems to me that getting rid of chained imports is not a big sacrifice
- N: what is the different between chained imports and pub use of modules?
- B: it seems like that would be forbidden
- P: can you be more specific?
- N: I meant something like this:
    mod a {
        mod b { fn c { ... } }
        pub use a::b::c;
        mod c { ... }
    }
    mod x {
        use a::c;
    }
- N: it seems ok to me that pub use has to be more concrete about what namespaces it re-exports
- *room is finished*
