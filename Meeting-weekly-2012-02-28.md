## Attending

Graydon, Niko, Jesse, Brian, Patrick, Tim, Marijn, Dave

## C types

* Graydon: maybe should gather agendas in advance in the future
* Graydon: looking at stuff nominated for 0.2, wanted to release in March, and we've done a fair amount of interesting work in past two months; nominate stuff we can get done in next week or two?
* Graydon: wondering if we're doing the right thing about types; some of our types are named same as C types, but not the same semantics — e.g., int; expected that char will change, but int is a tough one
* T: how different?
* Graydon: difference is 64-bit programming model; you have minimums but not maximums; most systems define LP64 or LLP64; pointers are 64-bit but not int, which remains 32-bit; we don't do that, we define it the same as pointer
* Marijn: what was our rationale?
* Graydon: wasn't a really good rationale; hadn't realized how widespread LP64 was; kind of feel like 32-bit integers are this vestigial 20th century thing
* Graydon: but interoperability is suggesting to me this might not be the wisest plan
* Graydon: we're allowed to shadow type names now, so you could shadow int with libc, but that's scary
* Niko: agreed
* Graydon: any strong opinions? ugly corner but will probably be noticed by everyone
* Niko: personally think we should just match C; downside is we need a name for long
* Graydon: could be defined in ctypes module
* Niko: don't think normal Rust programmers will want it?
* Graydon: u64
* Jesse: long is a weird name
* Niko: c_uint_ptr_t or something; don't really care what the name is
* Patrick: does this mean int would be a typedef for i32 and uint for u32? might be kinda nice
* Graydon: currently type-compatible with either u32 or u64, right?
* Patrick: depending on platform?
* Graydon: yeah
* Patrick: seems strange to me, but orthogonal
* Marijn: agree that's strange; recently broke a 32-bit build b/c I accidentally used an int
* Niko: happens to me all the time
* Patrick: happening all over; assuming type-compatibility of int with 64 or 32 and breaks on other platform; if weren't type-compatible at all, wouldn't happen; but would be fixed if it's 32 everywhere
* Graydon: yeah but that's what i32 is for; whole point of having int or uint type is to say "I want to be platform-specific"
* Niko: not sure that's what that means
* Jesse: or "I want this to be what's fastest on this platform"
* Patrick: type-copmatibility of int with others depend on platform is wrong
* Graydon: don't mind if its rep is platform-dependent as long as it's considered a disjoint type?
* Patrick: yes
* Niko: yes
* Jesse: yes
* Graydon: that's interesting; I'm fine with that too
* Patrick: if we make int always 32-bit then there's no problem with compatibility
* Graydon: would prefer if you want 32-bit that you just say 32-bit
* Graydon: I'll continue to play around with this
* Niko: fact that we added a lint pass for this error suggests something's wrong
* Dave: still concerned that saying "int" means "this is the way to do integers" and will have portability bugs
* Graydon: depends on their background, C programmers expect platform-dependence
* Graydon: one other thing to watch out for: the character type; char in C is not even defined as 8 bits, but happens to be the case; there are three character types: char, unsigned char, and signed char
* Graydon: unlike others, char is not signed or unsigned by spec, it's platform-specific and varies
* Marijn: when does Rust code care what the signedness is of a C char?
* Graydon: probably doesn't care substantially, just matters what I name the types; we'll wind up with three types
* Marijn: why not say Rust characters are always signed (or unsigned, whatever)?
* Graydon: Rust characters are u32's
* Niko: Marijn is saying, we can just say char in C is unsigned char; I kind of agree
* Graydon: if you define a local variable in terms of the underlying C type
* Jesse: first security hole I ever found was in IRC when they used a character as an array index
* Graydon: my concern is figuring out, if we plan to make all ctypes types have names that are either unique to the ctypes module (like void, or long, or uint_ptr); if we make int identical to Rust int where Rust int is identical to C int, then it's fine; but what about type char? cchar, cschar and cuchar?
* Jesse: why do we need a name instead of e.g. u8?
* Niko: nice to have the C names so when you're porting
* Dave: my gut reaction: distinguish in the name (with "c" or whatever) that it doesn't match, and don't bother with a distinct type if it matches perfectly
* Graydon: ok
* Jesse: wish we didn't null-terminate strings

## Mutable variables

* Niko: I've been looking at the local variables being mutable by default, I've been tagging local variables as mutable
* Jesse: could it just be "mut" instead of "let mut"?
* Niko: we decided not to
* Niko: want to check with everyone, can I push this?
* Niko: most variables are immutable, gut feeling is something like 1/3 but I don't have numbers; a lot can be rewritten to be mutable without much trouble (e.g. using vec_map instead of accumulating into a vector)
* Niko: don't implicitly capture mutable locals in a boxed closure; fact that it's copied becomes relevant, would rather you write copy explicitly
* Niko: people fine with that?
* Marijn: that's an improvement
* Graydon: I'm ok with that
* Patrick: yeah
* Marijn: did you find any code that implicitly copies mutable locals?
* Niko: haven't done that part yet, will let you know
* Niko: ok, I'll keep working on it; just something I'm doing in the background while waiting for long compiles
* Graydon: you might want to permit at first
* Niko: you can already write it today
* Graydon: ok, just start landing; merge conflict will bite you; do one file at a time
* Niko: good idea; I'll keep the check unlanded until we've resolved all merge conflicts
* Niko: declare variable mutable but don't mutate: error, warning?
* Graydon: not an error, maybe lint
* Patrick: agreed, like when Java yells at you for catching impossible exceptions

## Iteration

* Niko: not terribly happy with status of iteration in Rust, neither with previous nor with my attempts
* Niko: iteration library is close but not quite there
* Niko: thinking: don't know whether we should keep iter library, minimal version, or what
* Niko: before I get to that, one thing: want to re-base on a breakable version of iteration
* Niko: basically return bool whether you want to keep iterating
* Niko: break/cont would become sugar for ret true/false out of block; ret itself not allowed (b/c it doesn't do what you might expect; doesn't return from enclosing function)
* Niko: is that generally agreed to?
* Patrick: as long as there are good error messages
* Niko: maybe we can tweak unification algorithm to take context information
* Jesse: I think we want break/cont to be type-incompatible with true and false
* Niko: Graydon thought true/false are simpler; it's also really easy b/c compiler already knows about true and false :)
* Jesse: yeah but I don't want them to write ret true
* Graydon: ret is rejected in block
* Niko: have to say break/cont
* Marijn: I've been rewriting loops to use iterator function, and not being able to return in the middle of the loop is really painful
* Marijn: maybe some secondary version that does allow returning out of would be nice
* Graydon: I confirm this myself; TCP breakage is real
* Dave: yay, I'm not the only one
* Niko: I haven't been bitten but I don't doubt it; but being unsure bites me
* Jesse: I would love break with value, with an option type
* Graydon: right, becomes enum instead of bool; but becomes expensive
* Dave: can you not make it cheap in the case where the type is option unit?
* Graydon: I recall the difficulty being combination between amount of material you have to pass being heavy, dynamic allocation; also can't control iterator itself using simple loop construct; it has to use an alt that it's constantly checking; iterator protocol becomes more complicated
* Graydon: that's why I like bool
* Patrick: let's try bool and see how it works out
* Niko: I'm fine with bool
* Niko: if we were gonna support ret, how? setjmp/longjmp?
* Graydon: two basic options: propagate via enum or efficient exceptions; if we get our unwinder really fast it's possible
* Niko: almost all of Python, Ruby, Scala do this
* Marijn: weren't we fundamentally opposed to non-local returns? cuz there's a ton of awesome things we can do with them
* Niko: could do a simple effect system
* Dave: no such thing as a simple effect system...
* Patrick: I'm skeptical of making longjmp efficient in presence of RC, destructors, etc
* Graydon: yeah, nice thing about failure is idempotent; but if you're gonna catch and continue you have to be able to restore state
* Niko: having break would already be helpful
* Marijn: if we had good macros we could wrap the loop in a nice macro, so at least we could fake it; not as elegant but at least solves practical problems
* Jesse: are we expecting most loops will be inlined? if you don't use break, then you probably won't pay the penalty for the construct existing
* Niko: you're talking about break with value?
* Jesse: or just having break/cont at all
* Jesse: are we gonna have parallel iteration, parallel map?
* Niko: that remains to be seen
* Niko: can certainly build simple par_map like rustdoc has today
* Dave: still kind of on the drawing board
* Niko: I'll try to get this break-with-bool going
* Marijn: I'm kind of not having anything important on my plate; I wouldn't mind looking into it
* Marijn: is there an open bug?
* Niko: I'll find it and send it to you
* Niko: one more thing if that's all right

## Regions

* Niko: stuff with CCI is coming along, starting to actually work in stdlib
* Niko: hoping I'll be able to land it in some form soon
* Niko: would like to start experimenting with regions, implementing ideas I've been writing about
* Niko: is that cool with people?
* Dave: you can work w/o landing on master
* Niko: wouldn't put in 0.2 unless miraculously works
* Graydon: if by miracle it's a weekend of hacking, by all means; but if it's involved, hold off until 0.3, but certainly get started; I have no reservation
* Niko: ok, great
* Jesse: on the JS team, some concern about exchange heap: could result in different CPU's trying to write to memory close to each other, which can be bad for caches
* Niko: yes
* Graydon: yeah
* Niko: if you're using jemalloc and it's stuff you allocated, won't be an issue; if you send to someone else could be an issue; we may want to pad allocation sizes to try to avoid that
* Jesse: all exchange heap allocations?!
* Niko: possibly, at least for stuff you're sending
* Jesse: would you have different types for local-and-unique?
* Niko: no, probably not
* Graydon: kind of question you answer with hardware profiler
* Niko: yeah, wait till we have this problem, maybe allow an annotation
* Patrick: one thing relevant here: Niko, I know you and I have talked about this; I think we want to get vectors off exchange heap, esp. if they're not going to move around
* Graydon: yeah, we did something wrong with vectors; it's ok, they were already wrong
* Patrick: yeah, I don't know exactly what the solution is; my inclination is towards Java's approach; fixed-length vectors as primitive, can build dynamic vectors; overloading with impls can make dynamic vectors as convenient as fixed vectors; not sure exactly how that would look
* Niko: that's what we talked about in the no-copy proposal, which I hope to prototype with the region stuff; need to experiment with it
* Niko: Graydon, you mentioned "views" at one point, an idea I liked; there's some open design space
* Graydon: what idea?
* Niko: like slices as a second-class... well, first-class, but limited in how you can use them; can take a slice of the array without copying; kind of orthogonal, but would be nice to have
* Graydon: yeah, difficulty comes down to efficiency of reference counting the underlying vector; this is one of the cases where reference counts becomes noticeable; so either a tracing GC, or ability to publish to longer-lived region makes more sense
* Graydon: we'll have a better vocabulary for dealing with this when we have first-class region pointers; reasonable to talk about a task string-pool; people like to be able to intern
* Niko: right
* Graydon: read data in, then it becomes a constant for task life; useful concept to model in the language; GC's often do a special generation for pinned data
* Patrick: Niko convinced me that with regions it's quite possible to have dynamic arenas; could quite easily have task-local storage in a dynamic arena that you could pull from anywhere; just get a reference to that and put things in it; those pointers will stay alive for task lifetime
* Patrick: especially if we want modularity through tasks
* Graydon: I agree; just can't allow deleting stuff from TLS
* Patrick: yeah, dynamic arenas have no `delete`; you need placement `new` but no deleting
* Niko: that's a design goal of the regions stuff, to support that kind of use case
* Graydon: agree; will be easy to look at vectors once we have that vocabulary

## Bug sheriff

* Marijn: anything else?
* Graydon: couple small things but I can deal with most of them in bugs
* Graydon: one policy issue: float idea again of scheduling one day a week for cleaning out bugs in database; if there's an obvious bug you can knock down in a single setting, fixing improper tags, etc
* Marijn: should be staggered, one day per person per week
* Graydon: person on bug duty each day? I'll make a page, and people can sign up for days
* Niko: that sounds okay
* Patrick: could make it a sheriff thing; bug sheriff becomes build sheriff for the day?
* Graydon: don't really need a build sheriff right now
