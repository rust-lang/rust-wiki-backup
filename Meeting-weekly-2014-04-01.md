# Agenda 4/1/2014

- DST `Vec<T>` vs `~[T]` (acrichto) https://github.com/mozilla/rust/pull/13165
- intrinsics RFC (acrichto) https://github.com/rust-lang/rfcs/pull/8
- Virtual struct RFCs (larsberg) https://gist.github.com/jdm/9900569

# Attending

- pcwalton, larsberg, nmatsakis, cmr, brson, acrichto, pnkfelix

# Status

- cmr - tbaa, librustc cleanup
- pnkfelix: control-flow-graph, lifetime ICE's.
- acrichto: field privacy, tuple privacy, minor release things, send bound
- brson: 0.10, cloud services presentation

# DST Vec<T> vs ~[T]

- acrichto: There's a PR to remove ~[] vectors. It's brought up a few things. Vec<T> is growable. ~[T] is either frozen or an unfortunate fallout from DST. Lots of movement away from it because it was mainly used for growable vectors. Do we really want to be moving to Vec<T> for all of these? Or should we shun ~[T] everywhere? I'm curious: A) if we need to remove ~[] to implement DST, we should do it. Otherwise, it's nice to keep down churn and starting to use ~[T] where appropriate (e.g., by removing `push` and dealing with the fallout) would be good. Otherwise, people are just removing ~[T] all the time. Concretely, when you read an entire I/O Stream, ~[u8] seems right, whereas in this PR, it returns Vec<u8>, which is growable...
- pcwalton: I was removing ~[T] because niko was going to need us to remove all of old ~[T] for the DST change. If we can just remove `push` instead, that would be great.
- nmatsakis: I think we could just remove `push`, but I'm not sure. Maybe start that way and see if we have problems?
- brson: It sounds like everybody agrees that ~[T] is OK where it makes sense. Not just Vec<T> everywhere.
- pcwalton: Yes, this is just about the transition. 
- acrichto: There's also concern about every ~[T] comes from a Vec<T> And since the transition isn't free...
- brson: Should be.
- acrichto: Some people think we need shrink-to-fit.
- nmatsakis: I disagree.
- acrichto: Then, yes...
- brson: Can you shrink a Vec<T> independently?
- pcwalton: Ideally, Vec<T> should be rounding up to the nearest bin size in the allocator (b/c allocators usually have bin sizes) and should do its growth via that. Then shrink will almost never do anything. 
- acrichto: So we need to stop completely removing ~[T] from the standard libraries and instead think about whether we should return something frozen or growable.
- pcwalton: Definitely should not be returning Vec<T> unless you are sure someone should be able to push onto it.
- acrichto: And we'll add cheap ways between Vec<T> and ~[T]
- nmatsakis: From ~[T] to Vec<T>, what happens?
- acrichto: set capacity=length. Push might be expensive, but pop is cheap.
- nmatsakis: Could support pop on ~[T], in principle...
- pnkfelix: Doing the bin size allocation, doesn't the allocator then expose that and can't it be used for the capacity?
- pcwalton: Yes, but you'd need integration with the allocator. 
- nmatsakis: Could say you ask the allocator for its size.
- pcwalton: There is no API for that in C.
- pnkfelix: OK, it'd have to go in the allocator design.
- acrichto: Lots of comments back and forth in this PR. The opinion seems to be that we should favor one choice. Probably need mail to the mailing list about a shift to ~[T] for non-push things and Vec<T> otherwise.
- brson: We should send something laying out the full set of changes associated with this DST set of changes.
- nmatsakis: I should be writing a DST RFC with all of that info... we have a whole plan, but I haven't written it up yet. Maybe I should do so and send out an e-mail.
- acrichto: There will be feedback from the community, fyi.

# Virtual structs

- lars: Feedback from jdm about all the virtual struct proposals, wondering how they'd affect Servo and how they all play out with examples. He would like the RFCs to have examples of the servo use case that he sent out, and it would make it much easier to provide feedback about all the proposals.
- brson: What concrete action should we do? Backfill examples?
- lars: jdm has been contacting owners, nrc is following up on examples. Just want to make sure that all the propers of the virtual struct RFCs are aware that examples would be greatly appreciated to provide high quality feedback.
- brson: Nothing to do right now?
- lars: Right, but as we get farther on the proposals jdm will have more feedback on which one servo likes the best, etc.

# intrinsics

- acrichto: We have an RFC for it. It's been baking for a while and may be ready to merge. 
- brson: Any new criticisms?
- acrichto: Nope.
- pcwalton: Which ones here?
- brson: Rust intrinsics foreign function types, and removing it. We want the ABI to be the same as the Rust ABI and to be treated the same as regular Rust functions.
- nmatsakis: Aren't they just lang items?
- brson: Intrinsics can appear arbitrary numbers of times. So we'd either have to extend lang items or make intrinsics appear just once.
- nmatsakis: Why can they appear multiple times?
- brson: That's how foreign functions work.
- nmataskis: I prefer to say that intrinsics are lang items and ignore the body. I prefer to get rid of the notion of intrinsics and set them the same as lang items.
- pcwalton: Weird to have the body ignored... a foreign function as a lang item makes more sense.
- nmatsakis: With a Rust signature? OK, I guess I don't care. I prefer not to have it be a foreign function, since it's defined within that crate...
- pcwalton: Well, you also have to return something. So you'd have to put a fail in there or a zero, which is sort of a lie.
- nmatsakis: I'm mainly concerned with what the cleaner code path is. I'm fine with extern.
- brson: So, you want to make this change to use lang item instead of a new intrinsic attribute? I'll make that change today.
- cmr: The only change here is that the intrinsics attribute changes into lang=name_of_attribute?
- brson: Yes.
- acrichto: We forbid generics in foreign functions, though...
- brson: But they're allowed on Rust intrinsic foreign functions!
- cmr: There was an RFC a while ago about allowing generics there... I believe it was to expose instantiated generic functions as an extern "C" callback into a C library.
- acrichto: It was on definitions of foreign functions, not declarations of them.
- brson: So niko, you're okay with this exception? Generic is OK if intrinsic?
- nmatsakis: Yes, that's fine.
- acrichto: If you say extern Rust, you get a compile error. Could give you an error for everything that's not an intrinsic lang item and it'll be the same as it is today.

# strbuf

- pcwalton: Does everybody like the name StrBuf? For the Vec equivalent for strings?
- acrichto: It's all good.
- nmatsakis: We can change later if we want.
- pcwalton: I will not remove all ~str, just the string appending bits.
- acrichto: Just string pushing; can still append strings.
- nmatsakis: I think the analogy in a lot of strings with the buffer and you freeze for arrays and vectors, it's a nice correlation.
- pcwalton: Yes, wherever you use StringBuffer in C# or StringBuilder in Java

# inheritance

- cmr: What was undesirable in the original single inheritance proposal such that we needed a tree of them?
- pcwalton: Language was getting to Java-y :-)
- nmatsakis: Didn't address downcasting or thin pointers. Could handle with macros, but it was a pretty complex proposal that didn't even handle the whole thing. This was for the "traits extending structs bits"
- cmr: The http://smallcultfollowing.com/babysteps/blog/2013/10/24/single-inheritance/ one
- nmatsakis: That one predates the workweek one. It definitely didn't support a thin pointer to a node that also permits invoking virtual functions. Nor could you really downcast. We had a model with a Trait with methods for every subtype, but...

# release

- brson: It's going out Thursday.
