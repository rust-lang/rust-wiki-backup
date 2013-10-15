# Agenda 10/15/2013

- drop by value (acrichto)
- drop in static items (acrichto)
- removing rusti, #9818 (acrichto)
- priv keyword, #8122 (acrichto)
- Participating in GNOME OPW (tjc)
  - https://wiki.gnome.org/OutreachProgramForWomen/2013/DecemberMarch
- Semantics of multi-crate packages, as per https://github.com/mozilla/rust/issues/7240 (tjc)
- Improving the volunteer experience (tjc):
  - VMs that contributors can log in to to debug cross-platform build failures?
  - volunteer participation in meetings?
- Tim's next project (tjc):
  - rustpkg is beginning to demand less time; what's high-priority? (Parallelizing/integrating workcache with the compiler; GC; trans rewrite; something else?)
- (inner) attribute syntax https://github.com/mozilla/rust/issues/2569 (pnkfelix)

# Status

- brson: automation, windows toolchain, release planning
- tjc: fighting with Linux bots, task failure, multi-crate packages, version handling
- acrichto: morestack back on, removing std::io, thinking about static linking
- nmatsakis: multiple lifetime parameters, temp lifetimes, borrowck bugs
- pnkfelix: attribute syntax, ~[] benchmarks, gc

## remove rusti
- acrichto: how do people feel? It's tempting for users, but broken. Could be salvaged, but without a champion, it's a bad idea to have here. It should be on the 1.0 list or removed. It could be a standalone project.
- brson: how much work is it to keep up?
- tjc: it's already bitrotted
- brson: +1
- acrichto: will comment to remove it
- pnkfelix: removing rust was bigger than rusti

## attributes #2569
- pnkfelix: wanted to change the inner attribute syntax to remove the brackets. Wanted to use #! as the discriminant. Problem is that it conflicts with #!. Particularly on Windows, where it supports paths (e.g., c:\). So, maybe we should use a different one than !. Have a branch with a bunch of alternatives:

```
   # outer(attribute)  struct s;
   
   #! inner(attribute) ;
   
   #^ inner(attribute(caret))  ;
   #| inner(attribute(pipe))    ;
   ## inner(attribute(pound)) ;
   #: inner(attribute) ;


  #[foo = "bar"]    // outer attribute

   # foo = "bar"   struct s;  // outer attribute
   
   #^ foo = "bar" ;
```

   Maybe just get rid of id=attr for the outermost part of the grammar for attributes
   
   
- pnkfelix: I like #|, but too close visually to #!. kimundi likes #^, which is one that also grows well (e.g., `#<`, `#>`)
- jack: how about `#:`?
- acrichto: why are we dropping the `[]`s?
- brson: not necessary
- pnkfelix: maybe other point, which is that the attribute syntax is fairly general. nmatsakis and I were discussing that the lack of a terminal symbol makes it difficult to visually parse the scope of the attributes. Gist with the options:

   https://gist.github.com/pnkfelix/6992779

- pnkfelix: might keep around id=RHS, but we need an end delimiter.
```
   # foo = bar :  struct S;  // outer attribute with end-delimiter

  #[outer(attribute)]
  #^ inner(attribute) ;
```
- brson: did you prefer the brackets?
- pnkfelix: I liked them, but the community was strongly supportive of their removal.
- brson: if we had the brackets, we could keep the ! and say that your program can't call a program called `[`
- pnkfelix: or could keep the `[` for just the outer attributes and use the shorter version for inner attributes. Horrible bikeshed, but let's get it painted. maybe wait for pcwalton?
- nmatsakis: should be consistent between comments and attributes. hard to remember //#...;, etc. No strong opinion on brackets, though can see why it's visually appealing to remove them.  Maybe parens instead of `=`:

````
    this seems ok?
     #[path = "foo"]
     #path("foo")
```

- acrichto: it seems like all of the non-bracket proposals have drawbacks...
- brson: if we keep the brackets, should we still change the token to a caret?
- pnkfelix: issue with the `#!` isn't just that `[` can be something to run, but that apache on Windows treats files with that leading token in the file specially. What's the motivation for #! instead of #^?
- jack: whether we have brackets or not is not as big a deal as whether we have the ;
- brson: let's switch to caret and leave the brackets
- nmatsakis: change the comment syntax to ^ as well?
- brson: yes.
- pnkfelix: I had been pushing to keep the trailing ; for redundant information. If we keep the brackets, people will be quite upset about also needing the ;
- nmatsakis: probably too many. example:

```
fn foo() {
    #^[bar]
    vs
    #^[bar];
}
```

so compare with
```
foo () {
       #[bar="foo"] struct S;
       vs
       #^bar="foo";
}
```

- nmatsakis: even without the ;, it's a lot of tokens.
- pnkfelix: is keeping the brackets inner just for consistency? If we remove the brackets, switch to caret and use a ;...
- nmatsakis: keep consistent.
- pnkfelix: I give up on the trailing ; with []
- acrichto: that example is quite verbose
- brson: especially since it's the first thing you see in every crate
- nmatsakis: only case where we need crate inner attributes... file modules
- pnkfelix: also lets you copy/past
- nmatsakis: I like having them in function bodies themselves
- acrichto: do we need an outer syntax?
- nmatsakis: struct fields are an example
- acrichto: most attributes don't work on struct fileds, right? Just documentation.
- pnkfelix: on enum variants
- nmatsakis: not a lot of defined attributes for them yet
- brson: or @!, if we succeed in removing GC. And then we can put this off for months.
- nmatsakis: doesn't answer brackets vs. no brackets. @! with brackets afterwards still looks noisy.
- pnkfelix: are we just designing the least-work alternatives?
- brson: maybe a new type of attribute instead? So, remove the brackets and this one type of meta-item.
- jack: ... parenthesized version...
- pnkfelix: here's where it comes up in the code:

https://gist.github.com/pnkfelix/6992536
- brson: so now #^ATTR, right?
- pnkfelix: so inner would be `#^ATTR;`
- nmatsakis: top of the crate might look like, e.g.:

```
// I feel like it's unreadable without a space - nmatsakis
#^ license("MIT")
#^ ...

fn foo() {
  /*^
   * Doc comments look like this
   */
  #^ inline(never)
}
```

- brson: we have ~9 months to change our mind before we set it in stone
- nmatsakis: don't really like the look of the caret, but let's close it. And better than #!.
- jack: closure metadata literals use ^
- nmatsakis: added a doc comment as well above with the inner syntax...
- pnkfelix: how do we distinguish them?
- nmatsakis: with a !
- jack: like that the caret in doc comments is like a pointer at the thing we are documenting
- nmatsakis: on inner attributes, the semicolon looks great
- jack: optional?
- nmatsakis: mandatory everywhere
- pnkfelix: on outers, the semi seems to apply the end of it
- acrichto: on an outer, a space between the ^ and word is note needed. but on the inner, you want them.
- pnkfelix: support any number of spaces (zero or more)
- nmatsakis: do we need to postpone this until we see the code?
- pnkfelix: I have a branch and will send the code around. You can use the pretty-printer and the parser. The pretty-printer will spit out the one you want depending on an environment variable.

https://github.com/pnkfelix/rust/compare/fsk-inner-attr-syntax-2569

## drop in static items
- acrichto: right now, we ICE. Do we even want to allow it? Obviously, not ICE, but should it work?
- pnkfelix: issue?
- acrichto: just for the ICE, not the broader issue.

https://github.com/mozilla/rust/issues/9243

- acrichto: wanted to make locks statically initialized. destructor deallocates that item. Seems rare, though, because it hasn't been blocking anyone.
- nmatsakis: but static is becoming more powerful over time
- acrichto: prevents `static mut Option<T>`. Not a problem unless you have unsafe code. And if the destructor does anything with resources, it probably has to be mutated at some point, so you needed an unsafe block anyway.
- nmatsakis: what does C++ do?
- acrichto: some interestingly-defined order of destructors at the end of the program?
- nmatsakis: can always make an unsafe static mut with an unsafe static pointer, init'd with NULL, then store in it later
- pnkfelix: seems more honest
- nmatsakis: it's more annoying to access, but then the language isn't promising you anything
- brson: seems like the "life before main" problem
- nmatsakis: let's disallow it
- acrichot: statics are becoming a bit too powerful. Maybe disallow and give a compiler error for now and revisit it later, if it even causes problems.
- nmatsakis, brson: agreed

## multi-crate packages (7240)
- tjc: rustpkg should support a package with multiple crates. if a package has multiple library crates, how are you supposed to name those crates if they are all visible outside the packages? a package ID corresponds to a single library crate. If we gave multiple IDs to them, they would not be in the same package anymore.
- tjc: proposal: only the library / executable crates at the toplevel will be usable by other packages. second, rustpkg will build them all but only install the user-visible ones. third, can only refer to a crate in the same package if it's top-level
- nmatsakis: what does the install point mean?
- tjc: whatever rustpkg puts into the build directory only the things required is temporary
- jack: the use case is the extra crates would only be for building the main crate
- tjc: or one of bjz's projects has the examples subdirectory. They just need to be built in-place
- acrichto: what about runtime dependence on a library? we'd blow it away and then just crash at runtime
- tjc: naming is the issue, then. so install everything, but not allow them to be named...
- acrichto: src/build & src/bar, with both libraries in there?
- tjc; src/foo and some crates & src/foo/bar with some crates. Saying the src/foo/bar should not be vibislbe except within foo, because it's not clear how to refer to it otherwise
- nmatsakis: what's an example of the src/foo/bar? I don't think of them as nested?
- tjc: examples, like jack said. or things that need to be run in-place.
- brson: does this only refer to what rustpkg does implicitly? Is whatever you want writeable in rustpkg.rs?
- tjc: do anything you want or do preprocessing and then call the rustpkg API calls. In the first case, you can lay your code out however you want. In the latter case, we will make the assumptions.
- jack: rust-http is the main use case we have right now. It has a subcrate it requires to build itself. There are at least 3 different crates in servo that have special things they do. We use python right now for all of them except for rust-http.
- brson: could script rust-http manually in the package.rs file, right?
- jack: yes. your point that making it easy to script that stuff is fine
- nmatsakis: aren't we set up like that in the rust repository today?
- brson: they're all individual packages
- jack: use case is to supply a package id to each of these. The package ids are tied to the repository
- brson: can also have local packages, not tied to repository
- tjc: if we'd had rustpkg from the beginning, we might have had a different repository layout
- brson: if we have a local path, we should refer to it e.g., src/libstd
- nmatsakis: what about people on the outside who need to pick them up? usecase in rust is a collection of related code, but instead of in separate git repositories in servo-style, they put it in a single repository
- tjc: sounds like using a script would work
- nmatsakis: do users have to deal with the scripts?
- tjc: package author would do it
- nmatsakis: nobody wants to manually install libraries...
- tjc: normally referred to by URL. but if custom logic is doing something in a special way and putting things in a special place, it has to be documented outside the system
- brson: can rustpkg not refer to packages by URL within a repository? I thought we could refer to subpaths within a repository
- tjc: should work. possibly not tested, but would be a bug
- nmatsakis: in that way, a package=a crate, 1:1.
- tjc: simplifies the use cases
- jack: for glfw-rs, so you couldn't do github.com/bjz/glfw-rs, would have to do github.com/bjz/glfw-rs/lib?
- nmatsakis: how do you distinguish what's the name of the pkg vs. repo?
- jack: no / in repo names on github
- nmatsakis: other repositories?
- brson: out of time
