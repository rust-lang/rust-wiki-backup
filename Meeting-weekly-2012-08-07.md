# Agenda

- Mode progress (nmatsakis)
- Intern talks on Thursday, 2pm ( https://intranet.mozilla.org/Summer_INT12#Intern_Presentations ) (notably, eholk and lkuper)
- Case classes (pcwalton)
- `let mut` syntax (pcwalton)

# Status Reports

- Sully: static typeclass methods going in today
- nmatsakis:
    - lint modes are in place
    - some snapshotting still required
    - will send e-mail explaining how they are in use
- intern presentations coming up (Thursday): lkuper, eholk 
- graydon: structured constants kind of work
- pcwalton: I used them in https://github.com/pcwalton/fempeg -- they work. Amazing.
- *general marvelment*

# Let mut

- "Let's do this first, it's short---just syntax" -- famous last words
- pcwalton: General feeling that it would be ok to just type `mut` rather than `let mut`.  Do people agree?
- graydon: I don't care
- nmatsakis: I don't care
- graydon: you now have to write fewer muts due to struct types
- nobody cares.

# Case classes

- pcwalton: Several components to it
- pcwalton: One idea is to merge enum variants and structs
- pcwalton: Enums are just a group of structs or other enums, essentially
- pcwalton: So my idea is to introduce some ways of declaring structs, corresponding to nullary, n-ary variants
    - struct foo;  ->  newtype'd unit
    - struct bar(a, b, c);  -> newtype'd tuple
    - struct zed { ... }  ->  
    - this allows us to say that an enum consists of "struct bodies" (struct decl minus the keyword `struct`)
    - various other stages:
        - make variants
        - allow nested
- graydon: what about using an enum as in C? `enum foo { a, b, c }`? will that still work? 
- pcwalton: yes
- sully: is this limited to a tree of refinements?
- pcwalton: yes
- sully: is there a fundamental reason for this?
- pcwalton: prefix property (common fields)
- bblum: but sometimes you just want to inherit the "disjuctive possibilities"
- pcwalton: I wanted to support common fields too
- bblum: are common fields and arbitrary refinements mutually exclusive?
- pcwalton: I think it's a separate feature
- nmatsakis: you can limit multiple inheritance to case with no fields
- pcwalton: another issue is the syntax, I wanted to have the nested enum syntax which limits you to a tree structure
- graydon: sounds like some of this should be discussed in the bug report, code examples to clarify the precise variant
- graydon: in particular it seems like we can get some of this but not all into the 0.4 release cycle
- pcwalton: my general theme has been to get all the new syntax into 0.4
- graydon: I'm all in favor of being syntax complete, at least keyword stable

# Repeated vector literal syntax

- pcwalton: I implemented `[expr, ..N]` but it looks... weird
- sully: why not just a macro? repeat!{expr, N}
- pcwalton: in some code it's common, and I think it's nice if it looks nice
- graydon: it's really really common as we don't zero memory
- pcwalton: mp2 decoders use a lot of 2 by 32 by 3 arrays that must all be zeroed
- graydon: TCP servers too, basically anytime you must initialize a C structure
- graydon: another thing is that if we use a macro the lexer will consume 64k zeroes for a buf
- eholk: I don't think we can write it as a macro anyway because of the number
- pauls: sure, just use `s!{s!{s!{...}}}`
- pcwalton: So what should we use? `, ..N` looks funny to me, what about `x` as an operator?
- lkuper: would `[1, 2, ..1024]` expand to `[1, 2, 1, 2, 1, 2, ...]`? 
- pcwalton: no, it must be `[0, ..1024]`
- pcwalton: is anyone really opposed to `x` as a contextual keyword?
- lkuper: why do we want to get rid of 
- eholk: what about "vector literal * integer"?
- pcwalton: what about `**`?
- nmatsakis: ok by me
- graydon: I have a preference for `..`
- various options proposed, none yet reached consensus:
    - 1 ** 1024 (exponentiation)
    - [1] * 1024 (can't use it within the fixed-length array, prevents overloading)
    - 1 x 1024 (contextual keyword)

## An aside on macros (shifted in time)

- graydon: dherman and I had a long talk last week to convince me that our macro system was normal and based on 30-year-old macro technology
- graydon: the name `macro_rules!` is complex, maybe we need a very simple `macro!` that's closer to CPP
# Brian speaks: module
- brson: I started changing mod to module but I didn't like it
- brson: a lot of letters in high concentration: big lists of modules
- pcwalton: I was thinking comma seperated?
- brson: I don't like that either
- *pasted module in the channel*
- general on-the-fenceness except for pcwalton who slightly prefers module
- brson: ok, I'm going to punt on this and work on other syntax changes
- graydon: one minor qualifier is that we are changing `use` to `extern mod` but `extern module` is relatively long
- pcwalton: all rust programs will begin with `extern module std;` which is close to `public static void main()`
- nmatsakis: let's just keep mod
- graydon: extern mod it is

# Semicolon rule for tail expressions

- nmatsakis: I prefer to keep the semicolon rule the way it is
- pcwalton: I am fine with that, I was surprised to see the mailing list response was unanimous, people sem to like it

# match check

- tjc: unanimous approval of removing match check in the bugs
- tjc: any objections?
- graydon: I don't care either way, you can make a macro `careless_match!` if you like

Fin.
