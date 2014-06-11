# Agenda 6/10/2014

* unsafe pointers (acrichto) https://github.com/rust-lang/rfcs/pull/68
* internationalization (acrichto) https://github.com/rust-lang/rfcs/pull/93
* pub use globs (acrichto) https://github.com/mozilla/rust/pull/14509
* stylistic lints (acrichto)
* remove f128 (acrichto)
* partial_cmp (acrichto) https://github.com/rust-lang/rfcs/pull/100
* size hints Reader Writer (brson) https://github.com/rust-lang/rfcs/pull/46
* libserialize rewrite (erickt) https://github.com/rust-lang/rfcs/pull/22
* older RFCs (nrc) - https://etherpad.mozilla.org/gaUpZqsJkH
* struct literals grammar (nrc) - https://github.com/rust-lang/rfcs/pull/92
* hygiene and lifetimes (pcwalton)
* `as Trait + Send` (pcwalton)

# Attending

zwarich, jbailey, aturon, acrichto, brson, cmr, pcwalton, luqman, erickt, niko, huon, kmc

# Status

* acrichto - rustdoc, facade polishing, crate id RFC
* brson - automation, administrative, playing with uutils
* nrc - dst - removing ExprVstore, coercing returned values
* cmr - slogging through libsyntax (still)
* aturon - landing stability index overhaul, finishing conventions draft, task API overhaul, COW prototyping
* pcwalton - unboxed closures, picking off P-backcompat-lang issues
* pnkfelix - dataflow atop flowgraph
* nmatsakis - #5527

# Action Items

* brson - close https://github.com/rust-lang/rfcs/pull/37
* acrichto - merge https://github.com/rust-lang/rfcs/pull/48
* brson - close https://github.com/rust-lang/rfcs/pull/57 with explanation
* brson - close https://github.com/rust-lang/rfcs/pull/33 don't want to expand use of MaybeOwned
* acrichto - update https://github.com/rust-lang/rfcs/pull/68 and get somebody to merge
* brson - close https://github.com/rust-lang/rfcs/pull/104
* brson - merge https://github.com/rust-lang/rfcs/pull/93
* acrichto - merge https://github.com/rust-lang/rfcs/pull/92
* acrichto - merge https://github.com/rust-lang/rfcs/pull/87
* brson - close https://github.com/rust-lang/rfcs/pull/46

# RFCs

(Looking at https://etherpad.mozilla.org/gaUpZqsJkH, starting at line 34)

# RFC PR 37

* nrc: Clarification, cmr?
* cmr: Unrelated, still an extension of this rules.
* niko: It's related in that considering implementations separate from traits doesn't make any sense in my model. I guess you could say "all crates either implemented or defined".
* brson: Niko, are you inclined to do this?
* niko: I am not.
* pcwalton: Neither am I.

# RFC PR 48

* niko: I removed all the controversial bits, and we agreed to merge it after I did so. Should I merge it?
* niko: It doesn't cover associated types or functional dependencies.
* brson: Any objecting to pushing this?

# RFC PR 57

* nrc: Lots of discussion, anyone opposed?
* acrichto: Uncomfortable, don't want to commit to something that's wrong, want an expert in this area.
* brson: Seems like something that can be done out-of-tree. If servo cares about it, they can work with it on their own for a while. At this stage, accepting major subsystems into the core libraries is pretty risky.

# RFC PR 22

* erickt: I've been working on this for a bit. One of the issues we have with the current de/serialization is that we can't really take a generic object like the Json enum we have and deserialize into it.
* niko: Is this specific to JSON?
* erickt: We're serializing into a specific thing that lets us know what to look for.
* niko: ...
* erickt: This prevents us from taking an arbitrary json blob and deserializing it either to a specific struct or a generic Json object. To deserialize into a generic structure, we need at least one level of lookahead to check if the next thing is a list. This changes decoder to produce a stream of tokens (as an iterator).
* erickt: I have a prototype that fixes this and, as of now, if we change how we encode specific enums, is currently 8% faster than today's serialization. Can gain another 8% (by doing something...)
* niko: RFC says it's slower, is that out of date? Makes a big difference to me.
* erickt: Lets us encode/decode all rust structures
* niko: The set of tokens you opted for is sequences, maps, and structs?
* erickt: You can think of everything as the primitives and sequences, a map can be a sequence of tuples, but there are speed advantages to including maps. Need less tokens.
* acrichto: This sounds very similar to an iterator.
* erickt: It is an iterator, but the RFC might be out of date.
* acrichto: Would this make the decoder trait go away, and you'd be parameterized over an iterator of tokens?
* erickt: Right now you need an iterator and some helper stuff, so you need a Decodable/Deserializable trait which can be a subtrait of Iterator. One thing I haven't thought through is that deserializing (producing an iterator of tokens) and serializing is also producing a series of tokens, you might be able to merge them and have one thing that would produce an iterator of tokens. I'm not sure what the advantage of that is yet.
* niko: Enum Start and Enum End, in the case of json, do they correspond to stylized json?
* erickt: No, just a token that says you expect a compound structure, and when that ends.
* felix: There is some unspecified details about what that means.
* erickt: Yeah, I need to take another pass on this RFC.
* niko: I'm in favor.
* erickt: I don't want to merge this quite yet.
* huon: Have you applied to languages other than json? csv or something?
* erickt: I've talked to burntsushi and dwrensha about csv and capnproto. I think this works for csv, but that we can't really do anything about capnproto. Want to port EBML to catch any edge case.
* brson: It'd be great to see more encoders.
* niko: It'd be nice to have a minimal decoder that is as-fast and as-compact as possible. It's what we want to encode rustc metadata. Doesn't really interact with this.
* erickt: The overhead with this is tagging every type as you go through...
* niko: That is a new overhead, a type used to say what it was expecting, but not it can't
* erickt: The current version that we have has 0 lookahead, with unexpected things being an error. Now we can do something when we see something unexpected.
* niko: This will add substantial overhead.
* erickt: I expected it to be slower, but LLVM does a very good job of optimizing the state machine
* niko: I was working with metadata before and it got larger. I wonder if there's a way to have optional tags.
* erickt: You only need these tags to do the handshake between the encoder and the type. What actually gets stored can involve fancy tricks.
* niko: I'd like to look at it and think about it.
* erickt: One of the tricks I have is that you can have a Decoder with an expect_int and returns Result<Option<int>, Error>, you can skip having your data encoded in a certain way.
* niko: Right, tagless data. I'd like to see more data on this.
* brson: Niko, are you ok moving forward with the RFC or do you want to see more design?
* niko: I want to see a refreshed variation. I really want to see optional tags.
* brson: Eric, your job is to appease Niko.
* cmr: This seems to have a symmetry with reflection.
* erickt: It'd be interesting to see if this could be merged with reflection.
* niko: You could imagine a decoder that walks over memory.
* cmr: Seems like it could make our reflection go away.
* niko: It'd be nice to have opt-in reflection.

# RFC PR 33

* brson: I strongly want MaybeOwned to go away.
* acrichto, niko: I agree!
* felix: Do you want it to be more primitive? The notion is an important one.
* acrichto: I don't have a good replacement, and I feel bad saying no to this based on my personal tastes.
* niko: I don't feel bad about it. I'd like to find a more general solution. I don't know if we'll find something, but I hope we do.
* niko: But I don't want to commit to MaybeOwned until we've exhausted the alternatives.
* someone: is this a place where aturon's "impl Foo" could step in?
* niko: I don't think so.
* acrichto: Sounds like we should close this.

# Hygiene and Lifetimes

* pcwalton: We have this bug (https://github.com/mozilla/rust/issues/12512).
* pcwalton: The problem is that lifetimes and identifiers are thrown in the same bucket, but not really want you want to do. Might have the parser keep some sort of state when it decides  ....
* pcwalton: jclements says "just put a tick in front of the identifier". I've implemented it and it works beautifully. Any objections?
* felix: There's ..... unicode escapes?
* pcwalton: There's no way to make an identifier starting with a tick in Rust code.
* huon: You can with syntax extensions!
* cmr, pcwalton: Don't do that.

# RFC PR 68, unsafe pointers (rename *T to *const T)

* acrichto: One of the more popular opinions is to remove * altogether and to have Ptr<T> and MutPtr<T> etc structs. I personally feel like *const and *mut is the way to go. The impetus is that *T is too easy to type and doesn't mean what it means with T.
* niko: Doesn't work with DST, need to switch between pointers and fat pointers.
* cmr: Structs would either need UnsafeDeref or an unsafe deref method. I think having pointer structs would be altogether miserable.
* niko, brson: I prefer the original suggestions.
* nrc: ???
* felix: I'm in favor of the original plan, but the comment thread raises two points that I haven't considered before. One POV is that this is clearly the right thing because you want the FFI signatures to look like it does in C. Another POV is that it doesn't matter what FFI looks like, since a generator should be making those, but what your unsafe Rust code looks like, and we shouldn't tie ourselves to C names for C concepts.
* pcwalton: I think the argument that will everyone will use a generator to write their FFI is just as invalid as the argument that Java's verbosity is unimportant since everyone is going to use an IDE.
* niko: I don't accept the tooling argument either. There was also ???
* brson: That sends an important signal that you don't know what you're looking at, you need to go learn something here.
* niko: I think it's valid in that I want to validate Rust signatures, ???, but I think *const and *mut is good enough for that.
* felix: I'm basically in the same camp.
* brson: Does this RFC offer any guidance on converting between C pointers and Rust pointers?
* niko: Out of scope for the RFC, but something we clearly need to address.
* brson: ???
* niko: Do you mean we could include (syntax? wording?) saying exactly what *mut and *const mean?
* brson: Yes, what the aliasing rules and who can read/write them?
* acrichto: We don't have any, but I could write some.
* ...
* acrichto: I will probably send ...
* ...
* acrictho: We will coerce &mut to *mut and & to *const.
* brson: ...
* niko: Let's take an hour and write some text (to clarify the above questions).
# RFC PR 104

* niko: Proposed methods to permit

```
let x: (T, T, T, T) = ...;
{
let y: &[T] = x.as_slice();
... y[3] ...
}
let (a, b, c, d) = x;
```

* acrichto: We are at liberty to change struct layout whever we want now. This locks us into a specific one.
* (details)
* pcwalton: I don't think it's a problem.
* brson: I feel pretty ambivalent about this. I don't know how common it is to coerce a tuple to a slice.
* niko: We can defer this, the methods I want are already there. Maybe we'll find a more general solution, it's easy to add at any point.

# i18n and l10n of format strings (RFC PR 93)

* acrichto: Remove some of the stuff, and enables us to have a better syntax that requires less escaping. We would only have two special characters, curly braces, that are escaped by doubling them ({{ and }}). No backslash escapes in format strings.
* brson: This closes off some future options.
* acrictho: It closes off having .... (something different escapes?) ... in format strings.
* niko: I like it.

# RFC PR 92

* nrc: Minor disambiguation of struct literal syntax.
* ???
* nrc: Gives us a nicer grammar and allows us to ???
* pcwalton: Sounds good to me.
* (mic issues)
* nrc: The idea is that it's a small simplification around how we parse struct literals. The issue is that we decide whether we are in a block or a struct literal by searching for a colon, which is kinda hacky, and blocks doing type ascription in a nice way. Disallows using struct literals in some places.
* (general agreement)

# `Trait + Send` (RFC PR 87)

* pcwalton: I want to put this to rest. I want to separate bounds from traits. Colon syntax for bounds has lots of problems, proposed using `+` but also has problems. I want to remove the restriction from things that come after `as` such that it accepts any type but *not* `+`. You'll need to parenthesize it.
* niko: Can we parenthesize `Foo + Bar` in general? e.g. &(Bar+Send)
* pcwalton: I suppose.
* niko: I'm in favor of this and possibly going further.
* acrichto: Does this ambiguity exist today?
* pcwalton: No, parens not allowed.
* acrichto: &Foo:Send+Share
* niko: I think that might be technically legal.
* pcwalton: I suppose it does exist today, I propose changing it.

# Size hints on Reader and Writer RFC PR 46

* brson: I don't have many opinions on this. This seems like something I've never seen before in any other IO library.
* brson: Not a lot of opinions, sorry Simon!

# Stylistic lints

* acrichto: We have a lot of lints, and a lot of them are for style. I get annoyed when they trigger, and the stdlib has a lot of allows for this.
* acrichto: I think most of this comes from C naming conventions being entirely separate from ours. Requiring repr(C) on structs somewhat alleviates this. I would like to have a single style lint rather than many, that we set to deny in certain places. Having to put these allows everywhere is annoying. Maybe I'm the only one that hit this.
....
* felix: I think we all have our own subset of the lints we dislike.
* zwarich: We hit this a lot in Servo, when wrapping platform APIs. It'd be nice to have a better way to opt out of this.
* kmc: Loadable lints are nice for this too. We could have a meta-lint that disables all style lints.
* acrichto: Maybe by-warning lints are not very useful.
* niko: It's a matter of taste. Most of them are us being persnickety.
* brson: We've generally been adding these so we have a consistent style. How do we have that?
* acrichto: Fairly recently in the past these went from allow-by-default to warning-by-default.
* niko: It seems unfriendly to warn people about certain things.
* brson: I do like having one lint that is just "the house style", rather than separating them.
* kmc: Please don't do big changes to the lint code before my refactor lands!
* erickt: If this is a problem with C functions, can we just allow anything for those?
* acrichto: It's a problem with more than functions, includes structs and code written in the platform style.
* felix: Do we have support for hierarchy in the lints ("lint umbrella")?  Or would the switch to "house style" imply removing fine-grained control of individual lints?
* acrichto: Probably throw out the fine-grained lints.
* zwarich: Might be a good idea to have sub-lints.
* kmc: I can think about how to do that in the new system.
* acrichto: Out of time.

