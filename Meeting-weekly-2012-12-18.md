## Agenda
 - Azita and John attending! Hello!
 - sadness of buildbot snapshots (graydon)
 - repr change / recent bugs (graydon)
 - tree burning?
 - 0.5 triage
 - Trait inheritance
 - &|x, y|
 - T:&static
 - self → Self
 - calling static methods inside anonymous traits without having to qualify the trait

Attending: Graydon, Felix, Patrick, John Clements, Azita, Brian

## Buildbot

- G: Discovered buildbot snapshots are running newer linux. Binaries don't work on older linuxes. If you make a snapshot, you need to copy the linux binaries from s3 to static.rust-lang.org.
- G: Two repr (reflection) modules in core. We just switched to the new one yesterday. Keep your eyes open for bugs.
- B: New failing reflection tests
- G: Will dig into that
- G: Tree burnination. Patrick investigating?
- B: Hard to know all the failures. Bots are currently showing reflection failures.
- J: What does it mean for tree to be green?
- B: Bot related...
- P: Should we hold the release? Working on path changes. Mostly running into problems with pipes compiler.
- B: Also found a problem with trait inheritance
- P: If I don't land the path changes a lot of code will break in 0.6

## trait inheritence

- B: trait foo : bar means only one thing presently
- B: in <T:foo>, also T:bar
- B: should also mean this on object types (merge vtbls) but doesn't
- B: The reason I started calling these constraints...
- B: we don't always know at impl-declaration time whether for all types that implement some subtrait there is also an impl for the supertrait because of generics
- B: You may be right that it's possible, but it seems hard, and we don't do it now
- P: let's see what haskell does, copy them
- G: wait, are you talking about: "impl <T:foo> ..."

```
// checking that all impls of Bar are Foo. Can it be done at impl declaration time?
trait Foo { }
trait Bar: Foo { }
impl<T> T: Bar {
    }
```

- G: ok, this is indeed perplexing. I agree we should consult haskellism.
- P: Haskell calls them constraints
- B: One other issue. How does inheritance work with use statements. When you import Bar do you also get the methods on Foo?
- P: I would say 'yes', but it only goes far. Doesn't really extend to static methods. You can't treat static methods on foo, Foo::f() as being on bar, Bar::f()
- P: For now lets require use statements for each trait, see how usable it is.
- G: Triage time!
- P: Can I go over some quick things.
- G: Yes

## Pointer sigils on closures

- P: Cant write sigils on lambda syntax, but sometimes you want
- G: Maybe the default should be &, not @. Why is @ default?
- P: It's the first one we impemented
- G: Let's review when Niko is here

## T:&static

- P: region bounds replaces `Durable` trait

<general agreement>

## self -> Self
- P: Changing the 'self' type to 'Self'
- B: So will both be keywords?
- P: No, 'Self' will just be a builtin type, 'self' will become keyword
- G: I'm ambivalent

<shrugs and agreement>

## Calling static methods from anonymous trait without naming trait

```
    struct Ball { ... }
    impl Ball {
        /*static*/ fn new() -> Ball {
            ... Ball::foo() ...
        }
        /* static */ fn foo() {
            ...
        }
    }
```

## triage

... most deferred, a few related to trait inheritence kept to investigate before release
