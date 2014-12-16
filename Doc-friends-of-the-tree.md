At the [weekly meeting](https://github.com/rust-lang/meeting-minutes)
The Rust Team likes to occassionally recognize people who have made
outstanding contributions to The Rust Project, its ecosystem, and its
community. These people are 'friends of the tree', archived here
for eternal glory.

## 2014-12-16 Gábor Lehel (glaebhoerl)

Gabor's major contributions to Rust have been in the area of language
design. In the last year he has produced a number of very high quality
RFCs, and though many of them of not yet been accepted, his ideas are
often thought-provoking and have had a strong influence on the
direction of the language. His [trait based exception handling
RFC][tbeh] was particularly innovative, as well that [for
future-proofing checked arithmetic][checked]. Gabor is an exceedingly
clever Friend of the Tree.

[tbeh]: https://github.com/rust-lang/rfcs/pull/243
[checked]: https://github.com/rust-lang/rfcs/pull/146

## 2014-11-11 Brian Koropoff (unwound)

In the last few weeks, he has fixed many,  many tricky ICEs all over the compiler, but particularly in the area of  unboxed closures and the borrow checker. He has also completely  rewritten how unboxed closures interact with monomorphization and had a  huge impact on making them usable. Brian Koropoff is truly a Friend of  the Tree.

## 2014-10-07 Alexis Beingessner (Gankro)

Alexis Beingessner (aka @Gankro) began contributing to Rust in July,
and has already had a major impact on several library-related
areas. His main focus has been collections. He completely rewrote
BTree, providing a vastly more complete and efficient
implementation. He proposed and implemented the new Entry API. He's
written extensive new documentation for the collections crate. He
pitched in on collections reform.

And he added collapse-all to rustdoc!

Alexis is, without a doubt, a FOTT.

## 2014-09-02 Jorge Aparicio (japaric)

Jorge has made several high-impact contributions to the wider Rust community.
He is the primary author of rustbyexample.com, and last week published
"eulermark", a comparison of language performance on project Euler problems,
which happily showed Rust performing quite well.
As part of his benchmarking work he has ported the 'criterion' benchmarking
framework to Rust.

## 2014-07-29 Björn Steinbrink (dotdash, doener)

Contributing since April 2013. Björn has done many optimizations
for Rust, including removing allocation bloat in iterators,
fmt, and managed boxes; optimizing `fail!`; adding strategic
inlining in the libraries; speeding up data structures in
the compiler; eliminating quadratic blowup in translation,
and other IR bloat problems.

He's really done an amazing number of optimizations to Rust.

Most recently he earned huge kudos by teaching LLVM about
the lifetime of variables, allowing Rust to make much more
efficient use of the stack.

Björn is a total FOTT.

## 2014-07-22 Jonas Hietala (treeman)

Jonas Hietala, aka @treeman, has been contributing a large amount of
documentation examples recently for modules such as hashmap, treemap,
priority_queue, collections, bigint, and vec. He has also additionally
been fixing UI bugs in the compiler such as those related to format!

Jonas continues to add new examples/documentation every day, making
documentation more approachable and understandable for all
newcomers. Jonas truly is a friend of the tree!

## 2014-07-08 Sven Nilson (bvssvni, long_void)

Sven Nilson has done a great deal of work to build up the Rust crate ecosystem,
starting with the well-regarded rust-empty project that provides boilerplate build
infrastructure and - crucially - integrates well with other tools like Cargo.

His Piston project is one of the most promising Rust projects, and its one that
integrates a number of crates, stressing Rust's tooling at just the right time:
when we need to start learning how to support large-scale external projects.

Sven is a friend of the tree.

## 2014-06-24 Jakub Wieczorek (jakub-)

jakub-, otherwise known as Jakub Wieczorek, has recently been working
very hard to improve and fix lots of match-related functionality, a
place where very few dare to venture! Most of this code appears to be
untouched for quite some time now, and it's receiving some
well-deserved love now.

Jakub has fixed 10 bugs this month alone, many of which have been
long-standing problems in the compiler. He has also been very
responsive in fixing bugs as well as triaging issues that come up from
fun match assertions.

Jakub truly is a friend of the tree!

## 2014-04-22 klutzy

klutzy has been doing an amazing amount of Windows work for years
now. He picks up issues that affect our quality on Windows and picks
them off 1 by 1. It's tedious and doesn't get a ton of thanks, but is
hugely appreciated by us. As part of the Korean community, he has also
done a lot of work for the local community there. He is a friend of
the tree. Thank you!

- Rust on Windows crusader
- Fixed issues with x86 C ABI struct arguments
- Fixed multiple issues with non-US locales

## 2014-03-18 Clark Gaebel (cgaebel)

This week's friend of the tree is Clark Gaebel. He just landed a huge
first contribution to Rust. He dove in and made our hashmaps
significantly faster by implementing Robin Hood hashing. He is an
excellent friend of the tree.

## 2014-02-25 Erick Tryzelaar (erickt)

- Contributing since May 2011
- Wrote the serialization crate
- Organizes the bay area Rust meetups
- Just rewrote the Hash trait

## 2014-02-11 Flavio Percoco (FlaPer87)

- Contributing since September
- Does issue triage
- Organizing community events in Italy
- Optimized the 'pow' function
- Recently been fixing lots of small but important bugs
 
## 2014-01-27 - Jeff Olson (olsonjefferey)
 
- Contributing since February 2012
- Did the original libuv integration
- Implemented our second attempt at I/O, first using libuv
- Ported parts of the C++ runtime to Rust
- Implemented file I/O for the newest runtime
- Last week published an article about file I/O on the Safari books blog
 
## 2014-01-21 - Steven Fackler (sfackler)
 
- Contributing since last May
- CMU grad
- Lots of library improvements, Base64, Bitv, I/O
- Rustdoc improvements
- Mut/RefCell
- std::io::util
- external module loading
 
## 2014-01-14 - Eduard Burtescu (eddyb)
 
- Contributing since October
- Working on the compiler, including trans
- Reduced rustc memory usage
- Optimized vector operations
- Helping refactor the compiler to eliminate use of deprecated features
- Cleaned up ancient code in the compiler
- Removed our long-standing incorrect use of the environment argument to pass the self param
 
## 2014-01-07 - Vadim Chugunov (vadimcn)
 
- Contributing since June
- Fixed numerous bugs on Windows
- Fixing broken tests
- Improved compatibility with newer mingw versions
- Eliminated our runtime C++ dependency by implementing unwinding through libunwind