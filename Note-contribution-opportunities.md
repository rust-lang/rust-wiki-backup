Archive of brson's suggestions for how to contribute to Rust.

# 2014/01/26

## Break up libextra [#8784]

Getting our library ecosystem in shape in critical for Rust 1.0. We want 
Rust to be a "batteries included" language, distributed with many crates 
for common uses, but the way our libraries are organized - everything 
divided between std and extra - has long been very unsatisfactory. 
libextra needs to be split up into a number of subject-specific crates, 
setting the precedent for future expansion of the standard libraries, 
and with the impending merging of #11787 the floodgates can be opened.

This is simply a matter of identifing which modules in extra logically 
belong in their own libraries, extracting them to a directory in src/, 
and adding a minimal amount of boilerplate to the makefiles. Multiple 
people can work on this, coordinating on the issue tracker.

## Improve the official cheatsheet

We have the beginnings of a 'cheatsheet', documenting various common 
patterns in Rust code 
(http://static.rust-lang.org/doc/master/complement-cheatsheet.html), but 
there is so much more that could be here. This style of documentation is 
hugely useful for newcomers. There are a few ways to approach this: 
simply review the current document, editing and augmenting the existing 
examples; think of the questions you had about Rust when you started and 
add them; solicit questions (and answers!) from the broader community 
and them; finally, organize a doc sprint with several people to make 
some quick improvements over a few hours.

## Implement the `Share` kind (#11781)

Future concurrency code is going to need to reason about types that can 
be shared across threads. The canonical example is fork/join concurrency 
using a shared closure, where the closure environment is bounded by 
`Share`. We have the `Freeze` kind which covers a limited version of 
this use case, but it's not sufficient, and may end up completely 
supplanted by `Share`. This is quite important to have sorted out for 
1.0 but the design is not done yet. Work with other developers to figure 
out the design, then once that's done the implementation - while 
involving a fair bit of compiler hacking and library modifications - 
should be relatively easy.

## Remove `do` (#10815)

Consensus is that the `do` keyword is no longer pulling its weight. 
Remove all uses of it, then remove support from the compiler. This is a 
1.0 issue.

## Experiment with faster hash maps (#11783)

Rust's HashMap uses a cryptographically secure hash, and at least partly 
as a result of that it is quite slow. HashMap continues to show up very, 
very high in performance profiles of a variety of code. It's not clear 
what the solution to this is, but it is clear that - at least sometimes 
- we need a much faster hash map solution. Figure out how to create 
faster hash maps in Rust, potentially sacrificing some amount of 
DOS-resistance by using weaker hash functions. This is fairly open-ended 
and researchy, but a solution to this could have a big impact on the 
performance of rustc and other projects.

## Replace 'extern mod' with 'extern crate' (#9880)

Using 'extern mod' as the syntax for linking to another crate has long 
been a bit cringeworthy. The consensus here is to simply rename it to 
`extern crate`. This is a fairly easy change that involves adding 
`crate` as a keyword, modifying the parser to parse the new syntax, then 
changing all uses, either after a snapshot or using conditional 
compilation. This is a 1.0 issue.

## Introduce a design FAQ to the official docs (#4047)

There are many questions about languages' design asked repeatedly, so 
they tend to have documents simply explaining the rationale for various 
decisions. Particularly as we approach 1.0 we'll want a place to point 
newcomers to when these questions are asked. The issue on the bug 
tracker already contains quite a lot of questions, and some answers as 
well. Add a new Markdown file to the doc/ folder and the documentation 
index, and add as many of the answers as you can. Consider recruiting 
others in #rust to help.