```
Agenda:
Method call syntax (nmatsakis)
Boxed iface syntax (nmatsakis)
unary move operator (nmatsakis)
borrowck (nmatsakis)
Error handling (graydon)
ARC (pcwalton)
CME typeclasses (pcwalton)
static functions (pcwalton)
# Autoderef'ing
- N: Autoderef by all steps
- G: As a user I think you would want is an early error on any ambiguity (instead of just picking the first one)
- J: Why can you set up an "impl for @T" at all?
- G: because it's a type
- G: Any automatic resolution will be counterintuitive
- G All in favor?
- <general agreement>
# Unary move operator
N: Proposal: move as a prefix operator
N: y = move x
G: way that C++ operator move/rvalue refs work
G: useful in that it allows you to give away an argument on the caller side
G: negatives: extra keyword; doesn't handle swap
G: in Limbo and Go, unary arrow <-
G: f(<- y)
Sully: we already have "move" as a contextual keyword
G: "<-" doesn't work as a kind bound
N: can use in structures too
G: currently done by last-use; uncomfortable with amount of implicit copy/move distinction
G: I'm totally comfortable with unary move, but I do like "y <- x" being the binary form as sugar for "y = move x"
P: sugar is nice
N: indifferent about the fate of <-
G: if <- goes, then <-> should go too
G: Can we implement swap as a function in the stdlib?
Brian: yes, unsafely
N: I think you can, yes
G: you need "forget"
N: no
G: anyway, you can do it
N: swap with the RHS being an rvalue is also very useful and the current <-> doesn't support that (swapping with null, etc)
B: are we planning on getting rid of last-use?
N: No. It's still nice to have it implicit sometimes.
Jesse: "move" is a weird keyword; what about ^?
Patrick: looks like grawlix
J: are we removing move mode arguments?
P: I would like to
N: I think it's possible that we don't need modes at all
J: it's nice to be able to see "i'm done with this var" at the call site
P: agreed
G: same with implicit mutation; always nice to see the & at the call site
G: you get a warning if it's copied
P: are we all generally on board with the no-copies proposal?
<general agreement>
P: Eric's experiences with parallel algorithms show that it's important
N: Have we reached consensus on a unary move keyword or function?
G: I think the consensus is that everyone is open to adding the keyword or the function
# Boxed iface syntax (N)
N: Right now "value as iface", e.g. "x as add::<T>"
N: Proposed iface(val)
N: Motivation is to avoid repeating the name of the iface
P: I like repeating the name of the iface
Syntax when you want to create a sendable iface? "iface~(x)"?
Looks like a function / value constructor, which Graydon likes
N: table and move on
# Method call syntax
Tabled
# Bug week
P: It makes sense to take off some time to deal with resolve bugs
N: Resolve seems to have most of the errors
G: Been running into codegen
P: Also classes needs some baking
eholk (?): also metadata
P: Yes, maybe we can get rid of metadata entirely by serializing item ASTs and leaving the contents "blank" (e.g., empty fns etc). Then we only have serialized AST (with side table) as the sole kind of metadata.
tjc: what about side tables?
N: that's part of serialized the AST
P: in an ideal world we could resynthesize the side table information, but it's probably too slow
N: I think that would be slow
G: I think there is too much redundancy in what we serialize (in general), so removing metadata and replacing with AST seems like a good idea
G: I expect things like fast symbol lookup might be an issue
P: I think it's kind of a liability, it might be faster to just eagerly read in all decls for crates rather than doing things on demand.  
G: Hard to say.  We read in one big EBML chunk and then just jump around within a memory buffer.
G: Deserializing everything eagerly would be slow
N: We should be able to lazilly deserialize item-by-item
G: Important to preserve that as this helps to keep C++ slow (recompiling .h files)
P: So, if inlined AST is actually inside the metadata, maybe we can try to define our metadata in terms of #[auto_serialize] to avoid the undocumented schema problem
P: Instead of just serializing AST, we can just move over to auto-serializing structures (AST, as much as possible) and use that instead
N: #[auto_serialize] has proven robust to changes in the AST structure
All in general agreement.
# Borrow check
N: Almost ready to replace alias with borrow check
N: I ended up implementing an extension to purity, will send an e-mail to summarize
N: Can't recall how important that proved to be, though
# Error handling (G)
This week's thread: https://mail.mozilla.org/pipermail/rust-dev/2012-May/001847.html
G: Thread on the mailing list about error handling—someone asks why don't we have exceptions
G: Is everyone aware of the general design I have in mind?
G: Add global variables; add publish/subscribe const kind variables
G: Task-local dynamically scoped variable, like a Lisp *special*
G: Specials are common in older languages
G: Frequently used for things you thread through your entire program: stdin, stdout, "context variables"
G: Error handler is a function you can call that either fails or returns a value to recover
J: How do you know what type the error handler is to return?
G: Error handler has a static type, e.g. handles_file_not_found_error() : handler<T> where T is a fixup value; e.g. bool, or unit, or an open file
G: Like a condition system, intended to handle the error at the call site
P: haven't used them enough to know how useful they are, very different
G: none of us here has any real experience using condition systems like this, but I've read a lot of folk literature that suggests they work well
G: end result of my research: nobody has a scheme that anybody really likes
Jesse: still confused, might need an example of how this might work
Jesse: what does the thing that might invoke the handler look like?
G: normal code, like unchecked exceptions, nothing in the signature itself indicates what handler it might make use of
pauls: no special keywords, no language support, right?
G: right, just global variables
P: canonical example is a a repl: open "foo.txt", gives you a prompt allowing you to edit the filename if foo.txt doesn't exist
P: also useful for compiler errors, where exceptions are a really poor fit (need to continue compiling)
N: result types are pretty ok when you do want to back out
P: monadic sugar might be good for those cases
P: Step.JS has result::chain take a series of functions
N: a list of closures doesn't really support closures with 
N: I'm a bit skeptical about global variables
B: I would perhaps rather build globals into the library rather than language
P: dynamically scoped things don't necessarily require globals
G: true, cheap trick is that every item has a unique address in memory, so it can key off address of a const
G: does anyone mind if I try to reduce endless bikeshedding on this topic by saying this is the direction we want to go?
P: let's open an issue on the condition system
# Atomic Reference Counting (shared immutable data)
P: needs a better name for "atomic reference counting", so as not to be confused with "automatic reference counting"
P: want to be sure that everyone is on board with this since it was added with minimal people present
G: I was terrified to see it until I saw the `const` kind
P: minimal way to get data sharing, the thing to avoid is shared mutable state---we have no shared, we now have shared immutable too
G: I thought we should build this, but only with deep immutability
G: proposals for a better name for ARC, then?
# CME Typeclasses
P: I just wanted to get a straw poll on how people feel about this idea
P: I want to enforce coherence: one impl of a typeclass per type
P: This makes it easy to use with hashtables and so forth
P: The basic idea is to syntactically restrict where impls can go
Brian: does this prevent duck typing?
P: It may prevent duck typing (that is, implicit impl of an interface)
Brian: wouldn't having duck typing allow you to implement any iface anyway?
P: on a class only; classes can implement ifaces
G: can you point to the proposal, I'm not finding it
P: http://pcwalton.github.com/blog/2012/05/28/coherence/
Brian: also means you can't create impls for generic types?
P: You can create a new iface and then create implementations on any types you want
P: anyway, let's table, just wanted to bring it up
# Constructors: static functions, record-like syntax
P: Constructors are kind of problematic, partially because they must return a fully constructed object and so cannot fail
P: Plus they have a lot of boilerplate
P: There has been a proposal to use a record literal-like syntax to initialize the class (specify all fields at once) and then have static functions wrap this to act as constructors
P: Static functions also seem to address the issue of having impls where the type being impled over is not the receiver
P: they would simplify a lot of stuff, no need for typestate in ctor etc
B: if you had a ctor in a class in a module, you'd have to call Foo::Foo::Foo() 
B: anyway, I think it's probably the best idea so far though not perfect
P: being able to have failing ctors addresses an annoying issue in C++ and Java
P: what do people think?
G: haven't even used classes yet, don't expect to use them a great deal, don't have a strong opinion)
G: initialization by record literals is close to how it used to work?
G: ok with it but don't get why they can't live outside the class
tjc: how do you regulate access to private members?
G: but you don't have an instance?
P: reason is the initialization of private members, which I don't like
P: removes "new" fns and various boilerplate (self.x = x, self.y = y)
P: a lot of times you just want a nominal record anyhow
tjc: I think it would be good to get rid of the special case for self.x in typestate/liveness
P: separating allocation from initialization is "the right way"
G: one thing I wonder about is do we need the keyword "new" for placement new?
G: also, "drop" as a special component to the class still exists, right?
P: actually, given "coherent type classes", we could make drop and copy just special interfaces that you implement, rather than making it a "feature of the class"
G: as always, complex interactions
P: we could almost remove kinds
N: almost? but not quite?
G: a minor update about interfaces and reflection
G: I do have working a version of the refl. intrinsic module
G: compiler knows about it
G: lives in front directory somewhere (intrinsic.rs) which is basically #included'd into every crate
G: declares an interface type and an intrinsic function
G: will wind up with a few things, including hopefully drop/copy and some other stuff
G: allows you to visit types in a generic way, mechanism is drifting into focus right now
G: any objections to compiler knowing about the intrinsic module?
P: no, I like the idea of moving more stuff into there rather than having them be magic
P: I personally think kinds are best to think of as special interfaces
```