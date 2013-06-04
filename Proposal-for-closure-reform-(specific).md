This is a more specific, condensed version of [this] (https://github.com/mozilla/rust/wiki/Proposal-for-closure-reform#nikos-comments).

**Proposal**

My proposal is to **keep ~fn as part of the language**, and to **unify the way ~fn and &fn behave** so that it makes sense to continue using the same keyword for both of them. The idea is that there is one basic (second-class) closure type, written ```fn()```, which can be "configured" with a number of different options. These are:

* Where the environment packet is located. Write ```&fn()``` for environment on the stack, or ```~fn()``` for environment on the heap.
* Whether the closure references its captured variables or owns (moves/copies) them. *A function that references its upvars will own explicit borrowed pointers to those upvars.* A capture clause can be used for this, with "reasonable defaults" (borrow for stack closures, copy/move for heap closures) if the clause is omitted.
* Whether the closure can be copied. A closure can be copied if it has the ```fn:Copy()``` bound (or possibly ```fn:Clone()```). This and other kind bounds behave like "deriving". Note: stack closures which don't mutate their environment will satisfy ```Copy``` because their environment will be borrowed immutable pointers.
* Whether the closure transitively owns data in its environment (i.e., no captured & or @). This is indicated by ```fn:Owned()``` and is necessary for task spawning or putting functions in ARCs.
* Whether the closure can be called more than once. ```once fn()``` means the closure is consumed when it is called, and it may move noncopyables out of its environment.

So, the grammar for function types is simply: ```{&|~|@}[once ]fn[:k1[+k2...]]()```

**Examples**

* Functions that can be shared between tasks are written (at least) ```~fn:Owned()```. A task body is ```~once fn:Owned()```. A function that can go in an ARC is ```~fn:Owned+Const()```.
* The argument to ```option::map``` is written ```&once fn(T) -> U```.
* The closure from "the case of the recurring closure" would have to be written ```&fn:Copy()``` (which can't be satisfied with something with a ```&mut``` in its environment).

**Notes**

* We can also make allocation explicit (and remove the need for closure type inference) by requiring the pointer sigil before heap closure values, so you'd write e.g. ```do spawn ~{ task body; }```, or ```return @|a,b| { ... };```.
* No function type may be promoted to an ```&T``` if we do dynamically-sized types. We could make this more obvious by putting the sigil after the keyword like we had before (```fn&```, ```fn~```). This would also make it more clear that once fns are consumed when called (```once fn&()```). Another possibility is the notation ```once &fn()```.
* If we want to make functions derive ```Clone```, we'll have to autogenerate a function for cloning the environment, and include a pointer to it in the environment packet (so it could optionally be promoted to a function without ```Clone```), like a vtable. This could optionally be extended (with more difficulty) to support deriving other traits, like ```Eq```.
* A stack closure that wants to mutate its upvars has to dereference the borrowed pointer to them, such as ```let mut x = None; do something |val| { *x = Some(val); }```. When making method calls, autoderef will make this look the same as before; otherwise, a user who tries to do "the familiar thing" (i.e., write ```x = Some(val)```) will get an easy-to-understand type error. This proposal could instead be done with the "implicit reference" scheme of today, but then stack closures will only be copyable if they also satisfy ```Const```.

**Advantages**

* Kind bounds on the environment make putting functions inside ARCs possible.
* Stack closures can be copied if they satisfy ```Copy```.
* Task spawn is the same as always, "feels like the language is built around it", with no extra special-case syntax.