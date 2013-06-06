## Goal

The goal is to be able to express the *effects* that a function may (or may not) have in a static way that the compiler can check. Possible effects include task failure, code marked "unsafe", garbage collection, dynamic allocation in general, file I/O, nondeterminism, task rescheduling, and mutation outside of one's own stack frame. There could also be the ability to add custom user-provided, domain-specific effects (that don't depend on language primitives).

To be feasible for Rust, an effect system will have to be:
* **Unobtrusive**. A user who doesn't care about effects in their code should not have to write any effect annotations.
* **Lightweight**. When used, effect annotations should be clear in meaning and not need to refer to other effects which aren't relevant.
* **Polymorphic**. The system must express a higher-order function's effect depending on the effect of its argument closure. Otherwise that function would have to be assigned the top effect which would make it impossible to use anywhere any effect is restricted.

(huh, those three bolded words above are all the same number of letters. clearly that means this is a good proposal, no?)

## Syntax

**Grammar**

I pretty much don't care about what the syntax ends up being. Here I propose one possible syntax to show that there's at least one feasible way to do it. Feel free to suggest anything you think is prettier.

The only keyword I would add is ```effect```, used for declaring the name of a new effect domain. For example, this would appear in the stdlib:
```
#[lang="effect_fail"];
effect Fail;
#[lang="effect_unsafe"];
effect Unsafe;
```
After that, annotations on functions can refer to the declared effect. I would reserve two annotation-keywords for the effect syntax:
```
#[might(Effect1, ... EffectN)]
#[wont(Effect1, ... EffectN)]
```
Note an alternate possibility would simply be ```#[effect(Effect1, ... EffectN, not(EffectP), ... not(effectQ)]```.

There would also be a ```trustme``` keyword, for overriding the compiler's opinion on whether a function has a specific effect. For example, ```#[trustme(wont(Fail))]```. (Maybe this would have to introduce the ```Unsafe``` effect, unless ```#[trustme(wont(Unsafe))]``` is also used?)

**Inference**

Effects will be inferred across function boundaries as a whole-crate analysis. By default, functions will need no effect annotations; their effect will be inferred as the aggregate effect of their code. Explicit annotations would only need to be used in the cases of:
* Forbidding an effect that would otherwise be legal
* Declaring a function to have an extra effect that it doesn't actually have ("introducing an effect")
* Masking an effect that the compiler can't prove won't happen

Explicit annotations won't override inferred effects; rather, they'd modify the "default inferred effect" just by what is mentioned.

**Using annotations**

Effect annotations could appear in front of function definitions, function types (trait declarations, arguments, variables, storage locations), and potentially at the start of files or in front of module names in crate files. Some examples follow.
```
#[might(Reschedule)] // "fail" effect is inferred
fn pipes::recv(...)

#[might(IO)]
fn tmpdir() -> os::Path

#[trustme(wont(Fail))]
fn divide_unless_zero(x: int, y: int) -> Option<int> {
    if y == 0 { None } else { x/y }
}
```
Annotations on higher-order functions will also be able to refer to named arguments, perhaps with an extra sigil:
```
#[might($helper)]
fn foo(helper: fn()) {
    // FIXME: Call helper in the future.
    // Users shouldn't rely on helper not being called.
}
```

## Potential use cases

The most obviously useful reason to have effects is that currently destructors can leak memory if they fail when a task is already unwinding ([#910] (https://github.com/mozilla/rust/issues/910)). With effects, we could write:
```
#[lang="Drop"]
trait Drop {
    #[wont(Fail)] fn finalize(self);
}
```
A "fantasy" reason is that, with the old borrow checker (where ```&mut T```s were copyable, and ```&mut T``` could be borrowed into ```&T``` only if the surrounding code was "pure"), effect inference would avoid needing to write ```pure``` explicitly on any function you wanted to call from such code, and ```#[wont(Mutate)]``` could be inferred.

Other speculative reasons include:
* Reasoning about concurrency nondeterminism ([#3094] (https://github.com/mozilla/rust/issues/3094))
* Preventing garbage collection in performance-critical code (such as the renderer thread in Servo)
* Preventing dynamic allocation or rescheduling in "atomic"-context kernel code (I hear we're running in ring 0 these days)
* Allowing users to reason about whatever arbitrary effects their own software might have

## Design notes

I think ```trustme``` only makes sense in conjunction with ```wont```, so ```trustme(wont(...))``` would actually be ```trustme_wont(...)```.

If you wanted to express the entire effect of a function, rather than just modifying the default inferred effect, ```Anything``` could represent the "top" effect. So for example ```#[wont(Anything)] #[might(Fail)]``` would mean failure is the only effect this function could ever have; and ```#[might(Anything)]``` would mean this function couldn't be called from anywhere any effect is restricted.

Functions that corecurse will (of course) have to have the same effect. The analysis will treat strongly-connected components in the call graph as a single unit.

**Possible issues**

Data structures with function pointers inside could be a big challenge. This could mean a tuple of two closures, or a struct with function pointers inside, or an existential trait package with a vtable. For structural types like tuples, inferring effects within the type might be possible, but for "nominal" types, there is a cross-crate problem: Suppose you have a ```trait Foo { fn f(); }```, and everywhere the trait is instantiated (I mean ```value as Foo```, not just ```impl Foo```) the given object's ```f()``` doesn't fail. Then it would be safe to say the trait method is no-fail, except that would prevent a user outside the crate from instantiating a ```Foo``` with a failing ```f()```, which might be entirely possible. Instead, what I plan to do here is say that trait methods have the top effect (```#[might(Anything)]```), and the trait definition could be written with an explicit annotation if you wanted to reason about existential boxes.

There is something of a "library boundary discipline" risk here. Suppose Alice writes a library ```fn a()``` which happens not to fail, and Bob writes a ```#[wont(Fail)] fn b()``` that uses ```a()```. Later Alice, who doesn't care about effects, updates her library and makes it possibly fail. This breaks Bob's code in a way akin to changing the actual type signature of a function, except the "type" is inferred, which makes it more of a surprise. This downside is unavoidable given the desire to be unobtrusive in the common case.

EDIT to add: rntz pointed out that the trait method problem holds even when trait methods aren't called via vtables: if you have a typeclass-qualified type variable and want to call a trait method on it, that call has to have the ```Anything``` effect, even though which function will be known at compile-time. The reason for this is that somebody might later add a new impl of their own type with some weird effect, and the method call must account for the fact that new impls with new effects might be added later. Consider this: ```fn foo<T, U: Iterable<T>>(x: U, blk: fn() -> bool) { x.each(blk) }``` -- if we want to guarantee ```foo``` doesn't have some effect E, should the ```Iterable``` trait definition prohibit E in all possible impls of ```each```? If we want to call ```foo``` in a ```wont(IO)``` zone, we either have to monomorphize it by hand, or forbid anybody ever from doing some ```impl Iterable for FileSystemIterator```. It might be possible to resolve this issue by adding the ability to parameterize trait definitions over effect variables, but that's throwing unobtrusiveness out the window. How much of a problem this will be depends on how often effects are restricted in polymorphic functions. (EDIT again: It might be possible to infer trait (and struct) parameterization over effects, with one effect variable per trait method. Maybe even easier to do that than to come up with a nice syntax for writing it explicitly.)