Rust's primary documentation, including the tutorial, guides and manual, are on the Rust website.

* [Master](http://static.rust-lang.org/doc/master/index.html) - documentation for the 'master' branch of the git repository.
* [Release 0.9](http://static.rust-lang.org/doc/0.9/index.html) - documentation for the latest release.

Additional supplementary documentation is maintained here on the wiki.

## Other documentation

[[Releases|Doc releases]] - Links to current and old releases and documentation  
[[Detailed release notes|Doc detailed release notes]] - Further explanation of language changes  
[[Community libraries]] - A curated list of external libraries for Rust  
[[Rust for C++ programmers|Rust for CXX programmers]] - A cheat sheet  
[[Rusticon|The Rusticon]] - A glossary of terms commonly used in Rust and Rust tools.  
[The Periodic Table of Rust Types](http://cosmic.mearie.org/2014/01/periodic-table-of-rust-types)  
[[Unit testing|Doc unit testing]] - Writing tests and running them with the built-in test driver  
[[Using rustpkg|Rustpkg]] - Managing packages  
[[Using rustdoc|Doc using rustdoc]] - How to extract Markdown and HTML documentation from code  
[Package documentation](http://docs.octayn.net/) - Documentation for rust packages  
[[Continuous integration|Doc continuous integration]] - Test your GitHub-hosted packages with Travis CI  
[[Reading and writing files|Doc Reading and writing files]]  
[[Attributes|Doc attributes]] - The role of metadata in Rust code, with descriptions of many applications  
[[Packages, editors, and other tools|Doc packages, editors, and other tools]]  
[[Packaging Terminology|Doc Packaging Terminology]]  
[[Crate Hashes|Doc crate hashes]] - How Rust generates crate filenames, versions symbols, and why  
[[Computer Graphics and Game Development]] - Libraries and example projects  
[Pr&eacute;sentation du langage Rust](http://lea-linux.org/documentations/Rust) - Detailed documentation in French, with examples  
[[Building for Android|Doc building for Android]]  
[[Building for iOS|Doc building for iOS]]  

## Community

* IRC:

    *Note that to guard against botnet attacks we occasionally turn on moderation, disallowing
    unregistered users from joining or talking. You may need to [register](https://wiki.mozilla.org/IRC#Register_your_nickname) your nickname. Sorry for the inconvenience.*

  * [#rust on irc.mozilla.org][pound-rust] - Main Rust channel - general discussion
  * [#rust-internals on irc.mozilla.org][pound-rust-internals] - Rust compiler and library development
  * [#rust-gamedev on irc.mozilla.org][pound-rust-gamedev] - game development in Rust
  * [#rust-osdev on irc.mozill.org][pound-rust-osdev] - OS development in Rust
  * [#rust on irc.ozinger.org][pound-rust-korea] - Korean Rust community
* Mailing list [rust-dev]
* Reddit's [r/rust]
* User groups
  * [Rust Bay Area][rust-bay-area]
  * [Rust Korea][rust-korea]
  * [Rust Skåne][rust-skane]
  * [Rust 中文圈][rust-zh] (on Google+)

[pound-rust]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[pound-rust-internals]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-internals
[pound-rust-gamedev]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-gamedev
[pound-rust-osdev]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust-osdev
[pound-rust-korea]: http://chat.mibbit.com/?server=irc.ozinger.org&channel=%23rust
[rust-dev]: https://mail.mozilla.org/listinfo/rust-dev
[r/rust]: http://reddit.com/r/rust
[rust-bay-area]: http://www.meetup.com/Rust-Bay-Area/
[rust-korea]: http://rust-kr.org/
[rust-skane]: http://www.meetup.com/rust-skane/
[rust-zh]: https://plus.google.com/communities/100629002107624231185/


## Blogs

People sometimes write about Rust. Interesting stuff usually appears on [r/rust].

[Ben]: http://winningraceconditions.blogspot.com/
[Brian]: http://brson.github.com/
[Eric Holk]: http://blog.theincredibleholk.org/
[Erick Tryzelaar]: http://erickt.github.com/
[Felix]: http://blog.pnkfx.org/
[Graydon]: https://blog.mozilla.org/graydon/
[Niko]: http://smallcultfollowing.com/babysteps/
[Patrick]: http://pcwalton.github.com/
[Tim]: http://tim.dreamwidth.org/tag/research
[Zack]: http://blog.z0w0.me/

[r/rust]: http://reddit.com/r/rust

Some Rust classics:

* [Pointers in Rust: A Guide](http://words.steveklabnik.com/pointers-in-rust-a-guide)
* [A taste of Rust](https://lwn.net/Articles/547145/)
* [An overview of memory management in Rust](http://pcwalton.github.com/blog/2013/03/18/an-overview-of-memory-management-in-rust/)
* [Which pointer should I use?](http://pcwalton.github.com/blog/2013/03/09/which-pointer-should-i-use/)
* [Lifetimes explained](http://maikklein.github.io/2013/08/27/lifetimes-explained/)
* [Little things that matter in language design](http://lwn.net/Articles/553131/)
* [Operator overloading in Rust](http://maniagnosis.crsr.net/2013/04/operator-overloading-in-rust.html)
* [Embedding Rust in Ruby](http://brson.github.com/2013/03/10/embedding-rust-in-ruby/)
* [A first parallel program in Rust](http://blog.leahhanson.us/a-first-parallel-program-in-rust.html)
* [FizzBuzz revisited](http://composition.al/blog/2013/03/02/fizzbuzz-revisited/)
* [Ownership types in Rust, and whether they're worth it](http://tim.dreamwidth.org/1784423.html)
* [Reasoning about the heap in Rust](http://johnbender.us/2013/04/30/reasoning-about-the-heap-in-rust)
* [The Option Type](http://nickdesaulniers.github.io/blog/2013/05/07/rust-pattern-matching-and-the-option-type/)
* [How I got started hacking rustc](http://cmr.github.io/blog/2013/06/23/how-i-got-started-with-rust/)
* [Abstraction penalties, stack allocation, and ownership types](http://robert.ocallahan.org/2007/10/abstraction-penalties-stack-allocation_23.html)
* [Présentation de Rust 0.8](http://linuxfr.org/news/presentation-de-rust-0-8) - A very detailed article about Rust 0.8, in French!

## Presentations

* [Niko's linux.conf.au talk 2014](https://t.co/aaYgMqZprC) - Memory ownership and Lifetimes. January 10, 2014.
* [Bay Area Meetup, December 2013](https://air.mozilla.org/rust-meetup-december-2013/) - Alex Crichton - Channel Performance; Steve Klabnik - documentation; Chris Morgan - documentation; Luqman Aden - Minecraft chat client.
* [Bay Area Meetup, November 2013](https://air.mozilla.org/sprocketnes-practical-systems-programming-in-rust/) - Brian Anderson - Rust community; Patrick Walton - SprocketNES; Frank Denis - Rust at OpenDNS; Yehuda Katz - Embedding Rust in C.
* [John Clements, 10-minute talk (video)](http://www.youtube.com/watch?v=_KgXy7jnwhY) at SoCal PLS on Rust, Macros, and Hygiene. December 2013.
* [Felix's Codemesh 2013 slides](http://pnkfelix.github.io/present-rust-codemesh2013/fklock-rust-codemesh2013.pdf)
* Geoffroy Couprie's [Scala.IO 2013 slides](http://dev.unhandledexpression.com/slides/rust-scalaio/)
* Steve's presentation at RuPy 2013 "Nobody Knows Rust." [slides](http://steveklabnik.github.io/nobody_knows_rust/#/), video to come soon
* [Tim's presentation at OS Bridge 2013](http://opensourcebridge.org/sessions/970) - And [slides](http://opensourcebridge.org/wiki/2013/Rust%3A_A_Friendly_Introduction)
* [Niko's presentation at Northeastern](http://smallcultfollowing.com/babysteps/blog/2013/07/18/rust-presentation-at-northeastern/) - Slides only
* [An I/O system for Rust](https://air.mozilla.org/intern-presentations-reed/) - Eric Reed's intern presentation on I/O
* [Types of Types](https://air.mozilla.org/ben-blum-from-the-research-team-presents-types-of-types-in-rust/) - Ben Blum's intern presentation on 'kinds'
* [Default methods in Rust](https://air.mozilla.org/intern-presentation-sullivan/) - Michael Sullivan's intern presentation on default methods
* [A work stealing runtime for Rust](https://air.mozilla.org/2013-intern-todd/) - Aaron Todd's intern presentation on the Rust scheduler
* [Dave Herman's StrangeLoop 2012 talk](http://www.infoq.com/presentations/Rust)