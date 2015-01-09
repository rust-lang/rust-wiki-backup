# Where do we anticipate breakage during alpha?

## To *unstable* APIs:

Any unstable API may change, but the big changes we expect are:

* **Path reform:** The `path` module will soon be [reformed](https://github.com/rust-lang/rfcs/pull/474) to make use of DST, to give a better account of platform differences, and to more closely align with the [new C++ standard](http://article.gmane.org/gmane.comp.lib.boost.devel/256220). Implementation work is largely complete, and the rollout should occur soon after alpha.
    
* **IO reform:** An [overhaul of the IO APIs](https://github.com/rust-lang/rfcs/pull/517/) is being planned; please join in the conversation! These changes will be landing throughout the alpha cycle.

## To *stable* APIs/language features:

### Stable libraries

- *Integer auditing:* with the move away from int/uint, we need to establish formal conventions for integer type choices and roll them out. This will affect some stable APIs, but we will likely add some form of widening to limit the breakage.

- *Overflow semantics:* we plan to introduce [overflow checking in debug builds](https://github.com/rust-lang/rfcs/pull/560).

- *Send/Sync changes:* to allow threads to share stack data, we are considering [removing 'static from Send](https://github.com/rust-lang/rfcs/pull/458), probably accompanied by some shorthand for trait objects that should minimize breakage.

### Destructors

* *Unsafe destructors:* closing [some holes](https://github.com/rust-lang/rust/issues/8861) and ungating unsafe destructors may cause minor breakage.

* *Dynamic-drop restrictions:* We are switching to a scheme that uses [lightweight flags on the stack](https://github.com/rust-lang/rfcs/pull/320) to determine whether a value must be dropped rather than zeroing out memory. This implies some [small restrictions](https://github.com/rust-lang/rfcs/pull/533) on what moves are permitted relative to today.

### DST

- *Sized:* the default assumptions about whether type parameters are sized continue to be tweaked, and we hope to try an [inference scheme](http://smallcultfollowing.com/babysteps/blog/2014/07/06/implied-bounds/) for them, which may result in some minor breakage.

### Notation tweaks

- *`for` loops:* loops will change to take ownership of the iterator, in preparation for `IntoIterator` (or perhaps move directly to `IntoIterator`). This can be trivially worked around by using `by_ref`.

- *Closure types:* we hope to remove the `:` notation for closures in favor of [capture clause inference](https://github.com/rust-lang/rfcs/blob/master/text/0231-upvar-capture-inference.md).

- *Indexing notation*: either change to take the index by value (unlikely) or to include `IndexSet` (more likely).

### Traits and type system

- *Coherence:* we will continue to improve the coherence rules (e.g. [the orphan rule](https://github.com/rust-lang/rust/issues/19470)), although this will probably not break existing code but rather allow more code to work.

- *Subtyping:* some details of coercion and subtyping \[[1](https://github.com/rust-lang/rust/issues/18737), [2](https://github.com/rust-lang/rfcs/issues/281)\] are likely to be tweaked. We do not anticipate significant breakage here.


