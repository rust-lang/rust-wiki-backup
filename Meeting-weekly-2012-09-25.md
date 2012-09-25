# Agenda
- 0.4
    - legacy-modes setting (nmatsakis)
- deletion of shape code (pcwalton)
- exports / pub / priv (graydon)
- relicensing (graydon)
- infrastructure (graydon)
- tell us all about strange loop / Emerging Languages conference, someone!

# 0.4

Issues targeted at 0.4: https://github.com/mozilla/rust/issues?milestone=7&page=1&state=open

- Niko(?): Demoding: Want to use + instead of ++ for scalars. Have patch, but currently leaks. Should this target 0.4?
- Graydon: Yes. An important selling point of 0.4 is "If you don't put modes in your code, you don't have to think about them".

# Deletion of shape code

- pcwalton: new box annihilator works (surprising but great!)
    - last use of shapes is the log statement, should be easy to fix since `%?` doesn't use it
    - can kill shape code.  we lose cycle collector but we weren't using it anyhow.
- *general merriment*

# Exports / pub / priv

- graydon: implemented `pub`, `pub use` and so forth to mean "exports"
- graydon: what this means is that by default `export` keyword is meaningless
- graydon: you must add `#[legacy_exports]`
- nmatsakis: does it give a warning or anything if it sees the keyword `export`?
- graydon: no but not hard to do

# Doc sprint

- graydon: brson is doing great work in this direction

# Relicensing

- graydon: we seem to have reached a consensus with legal team
- graydon: systems library exemption on GPL2 means we don't need to worry about being used with GPL2 projects
- graydon: apache foundation and our own legal counsel agreed a terse 3-line ASL2 notice at the top of each file is sufficient.
- nmatsakis: including test files?
- graydon: unclear
- jesse: firefox tests (at least many of them) are public domain

# Infrastructure

- graydon: yesterday we got sign-off to bring buildbot infrastructure into production
- graydon: not for 0.4 but during 0.5 sometime 

# Last-use

- https://github.com/mozilla/rust/issues/2633#issuecomment-8680306
- tjc: I made things compile but hit perf regressions
- brson: what kind of regression?
- tjc: I noticed that type checking got 3x slower, for example
- tjc: either we now having copies that used to be moves...
- pcwalton: ...that seems likely...
- nmatsakis: ...especially because of the fact that we permit impl. copies of ~[T] in rustc
- tjc: ok

# Strange Loop

- dherman's slides: https://speakerdeck.com/u/dherman/p/rust
- there will be video: https://twitter.com/littlecalculist/status/250291385847123968
