# Agenda

- Intern time winding down (graydon)
- IO subsystem(s) : now is the time
- JEMalloc : also now?
- Method invocation ordering
- stage0 stdtest
- Default args / keyword args straw poll

## Attending

kmc, sully, nmatsakis, tjc, dherman, pnkfelix, graydon, toddaaro, brson, azita, ecr

## Interns leaving soon

- G: Interns leaving in 2 weeks. It's been a good summer. Horray!

## IO Subsystem

- graydon: New I/O has landed, those people who were chomping at the bit to make I/O work better, now is the time.
- graydon: Google libuv book and you will see a list of abstractions almost all of which we will want to wrap in the new runtime / I/O manager.
- brson: Jeff Olsen is working on binding files
- graydon: But there are many other things.
- brson: The issue with libuv on mac spawning extra threads has been fixed upstream, so we should upgrade

## JE Malloc

- graydon: New runtime does not have segmented stacks
- graydon: One of the reasons that jemalloc was off was that it broke segmented stacks
- graydon: Maybe we should turn on jemalloc now so that as we rewrite segmented stacks we can ensure they remain compatible
- graydon: You may have noticed there was a huge perf regression, all patched up now, I'm going to continue working on cycle time.
- brson: There is still a major regression on the nopt bots, I don't have a clue what that is besides just new runtime not being optimized

## Method invocation ordering

- N: I committed a fix to various aspects of objects but it has a change to how we resolve methods. Wanted to run it by people so they know how it works, discuss it.
- N: Change is that it gives priority to "inherent" methods. In particular if you have an object type or some type T with `impl T`, and you call a method defined on that, it will take precedence over the trait. Even if it has to do more autoderefs to get to it.
- N: So you can think of it as "we first look for inherent, then we go look for matching traits"
- N: If you don't do this, you get into infinite cycles between a trait and an object that impls it.
```
trait Foo { fn method(&self); }
impl Foo for @Foo { fn method(&self) { self.method() /* XXX */ } }
```
- N: If we don't make this preference, the call at XXX gets resolved to call the trait method, calls itself rather than the inherent method on self.
- G: So this is specifically to handle the fact that we no longer implicitly have @Foo impl Foo.
- N: yes, more or less.
- Cmr: what about derefing self before invoking the method?
- N: Doesn't work. Might work in DST world, but not here. You can't dereference an @Foo. You need an &Foo. `(**self).method()` would work, if you could do it, but you can't right now due to lack of DST.
- N: I think this rule is preferable anyways since it's the only way to invoke the inherent method whereas you can potentially (when we have universal method invocation syntax support) invoke the trait method manually.

# stage0 stdtest

- B: stage0 stdtest, now that the new runtime is merged, is fairly important because it enables quick turnarouns when testing changes to the runtime. It's also easy to break. It breaks pretty often. It's been suggested that we find some way to ensure it keeps working.
- N: It breaks because ... incompatible changes? Isn't it true that it breaks whenever we make an incompatible change to the runtime?
- B: not clear to me. I don't believe it has to break. It is true that we've traditionally broken it fairly often.
- N: just a matter of putting the wrong #[cfg()] 
- B: yeah
- G: Two aspects to this: what the user runs, what the bots run.
- N: can we just add it to `make check`?
- G: I'm ok with adding it to `make check`
- B: Ok, will open issue on it.
- F: What's the workaround? Change snapshots file?
- B: No, you can also build stage1 with the clean workspace and then you make your changes and rebuild stage1 with NO_REBUILD. Similar to building stage0 stdtest.
- A: how does this interact with C++ bindings?
- B: depends if you have to build new C++ code, gets ugly. If you only have to change C bindings on rust side, not so bad.

# Default args / kw args

- G: this has been in the background / conversation for a long time, finally turned into an issue. Wondering if anyone has strong feelings?
- G: https://github.com/mozilla/rust/issues/6973
- N: I lack strong feelings on it, not something I'd want to do in short term.
- N: Anecdotal / interesting thing: I worked with someone who did a study where name of formals and name of supplied were reversed. Like 'out' and 'in' were swapped etc. They found quite a lot of these, just as a simple lint. So it's ... plausibly a good error check.
- G: Syntactically there's not a ton of room.
- F: I don't find the FRU + structs example that bad.
- G: defaults is strictly a matter of saving yourself some typing.
- F: maybe there's some way to enhance the FRU syntax
- G: Just meant to poll on 1.0-horizon interest
- T N F: NO, far future at most! (at least kw-args)
- G: ok
- Cmr: note: doomlord has patch in progress for detault args. At least has it parsing and has AST.
- G: the thing with any kind of overloading is that it makes it a little harder to figure out which function gets called.
- N: Yeah, maybe so long as it's not ad-hoc overloading, just defaults on a single function body, it might not be so confusing. Would want to talk about with patrick around.
- G: Ok, we'll revisit with patrick around
