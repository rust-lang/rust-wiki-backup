You want to contribute to Rust? Frankly, you've made an excellent decision. Rust is developed by a small and inclusive community that takes *very kindly* to strangers. You are going to have a good time.

The way [many people][cmr] seem to get involved is by simply trying to write Rust code. Inevitably they hit a bug or missing feature, at which point they are compelled to [[write a patch|Note development policy]].

We coordinate through [email][rust-dev], [IRC][pound-rust], and the GitHub [issue] tracker.

[cmr]: http://cmr.github.io/blog/2013/06/23/how-i-got-started-with-rust/
[pound-rust]: http://chat.mibbit.com/?server=irc.mozilla.org&channel=%23rust
[rust-dev]: https://mail.mozilla.org/listinfo/rust-dev
[r/rust]: http://reddit.com/r/rust
[issue]: http://github.com/mozilla/rust/issues

## Hacking on Rust

Before hacking, read some tips on [[getting started|Note-getting-started-developing-rust]] and the [[development policy|Note-development-policy]]. The [[Notes]] section on the wiki contains general wisdom about the code base from other developers.

## Picking something to do

The simplest way is to look through the [issue tracker] for issues labeled with "E-easy", "I-enhancement", "I-wishlist", and/or "A-an-interesting-project". Unassigned bugs are always fair game. Assigned bugs that don't seem to be getting worked on actively can be fair game, but always check with the bug owner first in that case.

Outstanding bugs or feature requests in Rust often have a corresponding test in the [[test suite|Note testsuite]] that doesn't yet pass. One good way to jump into Rust development is to look for files in the test/run-pass directory containing the string `xfail-test`. Those tests all correspond to bugs that need to be fixed, features that someone needs to finish, or sometimes tests that need deleting.

The source is also littered with hundreds of comments marked with 'FIXME' and 'XXX'. In Rust, FIXME comments come with an issue number; sometimes these refer to issues that may be resolved easily.

Another easy way to get involved is to participate in bug triage. Every week we send out lists of bugs to contributors along with instructions on how to move them forward. If you would like to participate email Brian at banderson@mozilla.com.

If in doubt, ask on IRC. Somebody will surely have a task that needs to be done.

## Ideas for newbies

* Adopt a module in std and ensure all doc comments are provided, accurate, complete and follow the [[style guidelines|Note style guide]].
* Adopt a module and write unit tests for each feature. Bonus points for writing one that triggers a bug and reporting it.
* Examine the runtime library of language Xyz and build a table comparing features with Rust, identifying missing features.
* Proofread the tutorial, ensuring it works as described with the relevant release of the compiler.

Reddit has [more ideas still](http://www.reddit.com/r/rust/comments/1grj61/feed_us_some_low_hanging_fruit/).

[issue tracker]: http://github.com/mozilla/rust/issues
[contributing]: https://github.com/mozilla/rust/blob/master/CONTRIBUTING.md
