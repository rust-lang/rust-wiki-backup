## Goal

The goal is to be able to express the *effects* that a function may (or may not) have in a static way that the compiler can check. Possible effects include task failure, code marked "unsafe", garbage collection, dynamic allocation in general, file I/O, nondeterminism, task rescheduling, and mutation outside of one's own stack frame. There could also be the ability to add custom user-provided, domain-specific effects (that don't depend on language primitives).

To be feasible for Rust, an effect system will have to be:
* **Unobtrusive**. A user who doesn't care about effects in their code should not have to write any effect annotations.
* **Lightweight**. When used, effect annotations should be clear in meaning and not need to refer to other effects which aren't relevant.
* **Polymorphic**. The system must express a higher-order function's effect depending on the effect of its argument closure. Otherwise that function would have to be assigned the top effect which would make it impossible to use anywhere any effect is restricted.

## Syntax

**Basic grammar**

My original proposal for the syntax involved just using ```#[...]``` annotations, but for it to have proper polymorphism, effects will need to go inside type annotations in some cases.

First I would add the ```effect``` keyword. Its first use would be to declare the name of a new effect domain. For example, this would appear in the stdlib:
```
#[lang="effect_fail"];
effect Fail;
#[lang="effect_unsafe"];
effect Unsafe;
```
After that, annotations on functions can refer to the declared effect. To express the effect of a function (which will hopefully be inferrable in many cases), you write ```effect``` again in the function signature, after the return type:

```
fn a() effect(Effect1, ... EffectN)
```
This can be used to "introduce" effects, even if the compiler doesn't infer them.

There will be another keyword, ```wont```, for writing effect *assertions*: that is, asking the compiler to reject the function if its inferred effects include a forbidden one.
```
fn b() wont(Effect1, ... EffectN)
```
Higher-order functions can be reasoned about like this:
```
fn hof1(blk: fn()           ) wont(Fail)  // won't fail unless 'blk' fails
fn hof2(blk: fn() wont(Fail)) wont(Fail)  // won't fail; 'blk' can't either
fn hof3(blk: fn() wont(Fail))             // might fail, but 'blk' can't
```
There would also be a ```trustme``` keyword, for overriding the compiler's opinion on whether a function has a specific effect. For example, ```fn b() trustme(wont(Fail))```. (It might have to introduce the ```Unsafe``` effect, unless that one is also squelched.)

**Grammar for parameterized structs, traits, etc**

Data structures might be passed around with function pointers inside. Unless we want to say such function pointers could have the "top" effect, or prohibit storing effectful functions inside of these, we have to be able to parameterize structs, enums, and traits with effect variables. To distinguish from type and lifetime variables, I'll say an effect variable is written ```%e```. Effect variables can be bound by effect *literals* (always written "```wont(...)```") in a similar way as type variables can be bound by traits.

Closures are also parameterized. The functions in the above example are actually short for:
```
// won't fail unless 'blk' fails
fn hof1<%e            >(blk: fn() %e) effect(%e) wont(Fail)
// won't fail, and 'blk' can't fail either
fn hof2<%e: wont(Fail)>(blk: fn() %e) effect(%e) wont(Fail)
// might fail, but 'blk' can't fail
fn hof3<%e: wont(Fail)>(blk: fn() %e) effect(%e)
```
You shouldn't usually have to write effect variables explicitly, but if you wanted to restrict the effects of trait methods you would have to do something like:
```
trait Foo { fn foo(self); }

// can't be instantiated with a T whose SomeTrait impl methods might fail
fn something<T: Foo<wont(Fail)>> (x: T) wont(Fail) { x.foo(); }
```
which is short for:
```
trait Foo<%e> { fn foo(self) %e; }

fn something<%e: wont(Fail), T: Foo<%e>> (x: T) effect(%e) wont(Fail) {
    x.foo();
}
```
The syntax for structs/enums would be analogous. By default, only one effect variable would be used for structs and traits, and the inferred effect of a struct/trait would be the aggregate effect of all functions inside. But you could write additional ones explicitly if you wanted:
```
enum WackyLazyList<%e1, %e2, T> {
    Nil,
    // effects alternate, i guess? why would you want this? o_O
    Cons(~fn() -> T %e1,
         ~fn() -> ~List<%e2, %e1, T>),
}
```
You can, of course, also restrict or introduce effects in traits/structs with effect literals.

**Inference**

Effects will be inferred across function boundaries as a whole-crate analysis. By default, functions will need no effect annotations; their effect will be inferred as the aggregate effect of their code. Explicit annotations would only need to be used in the cases of:
* Forbidding an effect that would otherwise be legal
* Declaring a function to have an extra effect that it doesn't actually have ("introducing an effect")
* Masking an effect that the compiler can't prove won't happen

Explicit annotations won't override inferred effects; rather, they'd modify the "default inferred effect" just by what is mentioned.

If you wanted to express the entire effect of a function, rather than just modifying the default inferred effect, ```Anything``` could represent the "top" effect. So for example ```effect(Fail) wont(Anything)``` would mean failure is the only effect this function could ever have; and ```effect(Anything)``` would mean this function couldn't be called from anywhere any effect is restricted.

Functions that corecurse will (of course) have to have the same effect. The analysis will treat strongly-connected components in the call graph as a single unit.

## Potential use cases

The most obviously useful reason to have effects is that currently destructors can leak memory if they fail when a task is already unwinding ([#910] (https://github.com/mozilla/rust/issues/910)). With effects, we could write:
```
trait Drop {
    fn finalize(self) wont(Fail);
}
```
We might also want to forbid GC in destructors, as per (#6996) [https://github.com/mozilla/rust/issues/6996].

A "fantasy" reason is that, with the old borrow checker (where ```&mut T```s were copyable, and ```&mut T``` could be borrowed into ```&T``` only if the surrounding code was "pure"), effect inference would avoid needing to write ```pure``` explicitly on any function you wanted to call from such code, and ```#[wont(Mutate)]``` could be inferred.

Other speculative reasons include:
* Reasoning about concurrency nondeterminism ([#3094] (https://github.com/mozilla/rust/issues/3094))
* Preventing garbage collection in performance-critical code (such as the renderer thread in Servo)
* Preventing dynamic allocation or rescheduling in "atomic"-context kernel code (I hear we're running in ring 0 these days)
* Allowing users to reason about whatever arbitrary effects their own software might have

**Some other examples**

Effect annotations could appear in front of function definitions, function types (trait declarations, arguments, variables, storage locations), and potentially at the start of files or in front of module names in crate files. Some examples follow.
```
fn pipes::recv(...) effect(Reschedule) // "fail" effect is inferred

fn tmpdir() -> os::Path effect(IO)

fn divide_unless_zero(x: int, y: int) -> Option<int> trustme(wont(Fail)) {
    if y == 0 { None } else { x/y }
}

fn foo<%e>(helper: fn() %e) effect(%e) {
    // FIXME: Call helper in the future.
    // Users shouldn't rely on helper not being called.
}
```

## Design questions and problems

**Big worries**

There is something of a "library boundary discipline" risk here. Suppose Alice writes a library ```fn a()``` which happens not to fail, and Bob writes a ```fn b() wont(Fail)``` that uses ```a()```. Later Alice, who doesn't care about effects, updates her library and makes it possibly fail. This breaks Bob's code in a way akin to changing the actual type signature of a function, except the "type" is inferred, which makes it more of a surprise. This downside is unavoidable given the desire to be unobtrusive in the common case.

There are some ubiquitous idioms for which it's not clear whether they should be allowed in code where their associated effect is forbidden. For example, should ```assert``` have the ```Fail``` effect? If so, it would be impossible to use any asserting library calls inside a destructor, or else the library author would have to separate all their assertions out into helpers tagged with ```trustme(wont(Fail))```. But if we say ```assert``` has a free pass, then if a destructor does trip an assert, we end up with the same memory leak the whole analysis is designed to prevent. Other examples include: should debug print statements count as ```IO```? should cactus stack growth count as ```Allocate```?

Another problem, which might even be a show-stopper, is the fact that destructors inside garbage-collected memory can cause arbitrary (non-Fail) effects to happen anywhere that ```GC``` can happen. Consider:
```
fn x() wont(IO) {
    let _ = @(); /* triggers GC */
}
fn y() {
    let _z = @DeferredCloseFileHandle(...); x();
}
```
One completely-non-starter idea is to forbid putting destructorful data in GC boxes. I was also considering the possibility of runtime instrumentation, but all those involve blocking a subset of GC for a "surprise" amount of time, which I don't find palatable.

I think the best way to deal with this is to say that ```GC``` is actually the (almost-)"top" effect; namely, if you want to forbid any other effect (apart from ```Fail```), then ```GC``` is also automatically forbidden as well.

**A question about smart defaults**

When a function uses implicit effect parameters, if it also has a ```wont(...)``` assertion on the function itself, it's not clear whether that should automatically constrain the effect vars, or whether you should have to constrain them separately.

Phrased more intuitively, this is the difference between a function saying "I personally won't do X, but can't be held accountable for what you instantiate me with", versus saying "When you call me, this effect won't happen, period". The former is what we used to have with ```pure``` functions, but the latter is more what we want for masking ```Fail```. The way I presented above is the ```pure```-friendly way.

Consider what would be the "most concise" ways to write each variation in each scheme. In the ```pure```-friendly way, we have:
```
fn f1(blk: fn()        ) wont(X)  // won't X unless 'blk' X
fn f2(blk: fn() wont(X)) wont(X)  // won't X, and 'blk' can't X either
```
With this way, I fear it'd be too easy to screw up and allow the function to be instantied with something that could have an unexpected effect. The default for the other way is somewhat more verbose, though, as you have to use effect vars.
```
fn f1<%e>(blk: fn() %e) effect(%e) wont(X)  // won't X unless 'blk' X
fn f2    (blk: fn()   ) wont(X)             // won't X, and 'blk' can't either
```