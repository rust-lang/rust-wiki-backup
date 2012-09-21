This page covers releases in more detail than the bullet-point list given in the RELEASES.txt file in the source distribution, in particular focusing on _language level changes_ that will be immediately visible and/or disruptive to users trying to keep their Rust code compiling and working right between releases. It is intended to hold copied, cleaned-up versions of entries from [[Note development roadmap]] as they are completed, to help users plan migration on their own code.

## 0.4 September 2012

This version was focused on completing as many disruptive syntax changes as possible, including changing and reducing keywords, beefing up traits and removing classes, changing how imports and exports are handled, and moving to a camel case convention for types.

Borrowed pointers matured and replaced argument modes in some of the libraries. A number of new concurrency features were added.

### Camel cased types

We have a new convention that requires that types (and enum variants, which are not yet types) be camel cased. The entire core and standard libraries have been converted. We generally prefer that acronyms not be written with all caps, so e.g. the standard URL type is written `Url`.

There is a lint check for this called `non_camel_case_types` and it is configured to warn by default.

### Keywords

This release tried to narrow down the set of keywords and the current keywords are expected to be close to final.

Keyword changes:

* `iface` became `trait`
* `ret` became `return`
* `alt` became `match`
* `again` was removed in favor of reusing `loop` to continue, i.e. `loop { if foo { loop; } }`
* TODO

The current keywords are as follows.

```none
as, assert, break, const, copy, do, drop,
else, enum, export, extern, fail, false, fn, for,
if, impl, let, log, loop, match, mod, move, mut,
priv, pub, pure, ref, return, struct, true, trait, type,
unsafe, use, while
```

Notes:
* `be` is not a keyword, but is reserved for [possible future use](https://github.com/mozilla/rust/issues/217).
* `self` and `static` are currently parsed as contextual keywords, but are expected to not be keywords in the future.
* `export` will be removed in favor of `pub` and `priv` item-level visibility (see below).
* [`assert`](https://github.com/mozilla/rust/issues/2228), [`log`](https://github.com/mozilla/rust/issues/554), and [`fail`](https://github.com/mozilla/rust/issues/2232) are likely to be converted to macros.
* `drop` may [become a trait](https://github.com/mozilla/rust/issues/3061).

### Structs replace classes

Classes are undergoing a major overhaul and have been removed from the language, in favor of method-less `structs` combined with method-bearing `impls`. The new `struct` syntax is very simple:

```
struct Cat {
  name: ~str,
  tail_color: KittyColor 
}

let my_cat = Cat {
  name: ~"Morton",
  tail_color: KittyColorBlack
};

// Methods for all types are defined in impls
impl Cat {
  fn meow() { }
}

my_cat.meow();
```

There is no explicit constructor syntax. The current, and likely temporary, convention for constructors is to create a function with the same name as the type. In the future constructors will probably be defined with static `new` methods.

```
// Current convention for defining constructors
fn MyStruct() -> MyStruct {
  MyStruct {
    field1: initial_value_of_field1(),
    field2: initial_value_of_field2()
  }
}

let m = MyStruct();

// Potential future convention for defining constructors
impl MyStruct {
  // Here, the staticness of the function is indicated by the lack of
  // an explicit self-type, another in-progress change.
  fn new() -> MyStruct {
    MyStruct {
      field1: initial_value_of_field(),
      field2: initial_value_of_field()
    }
  }

  ...
}

let n = MyStruct::new();
```

The last remnants of classes, destructors are temporarily implemented with `drop` blocks on structs. In future releases destructors will be implementations of a `Drop` trait.

```
// How to write a destructor today
struct MyStruct {
  field1: Field1Type,

  drop {
    // Run destructory code here
  }
}

// How to write a destructor in hypothetical future-Rust
impl MyStruct : Drop {
  fn drop() {
  }
}
```

See also pcwalton's [proposal][minmaxclasses].

[minmaxclasses]: http://pcwalton.github.com/blog/2012/06/03/maximally-minimal-classes-for-rust/

### Interfaces extended to full traits

([#2794](https://github.com/mozilla/rust/issues/2794)) Traits are interface-like units of behavior that carry method implementations and requirements on the self-type; they can be used to achieve code reuse without introducing much of the pain that comes from conventional inheritance: they can be mixed in any order, can be declared independent from the type they affect, and are compiled independently for each type that implements them (indeed, will inline and specialize exactly as any other function will).

This work involved replacing the `iface` keyword with `trait` and supporting full method implementations within traits, not just signatures. The terms employed by Rust's OO system are therefore now `struct`, `trait` and `impl`. We believe this will be the final set of terms.

### Implementation coherence enforced

([pcwalton:impl-coherence](http://pcwalton.github.com/blog/2012/05/28/coherence/)) Previously each `impl` had a name and when a client called a method on an `impl`, the code dispatched-to was chosen based on the `impl` imported into the _client code_'s scope. This was chosen as a way to be unambiguous about selecting implementations -- a problem in any typeclass system -- but in practice it has been very confusing for users: many are unable to tell why a method can or cannot be seen due to the presence or absence of imports.

Now every type can have at most one `impl` defined for each `trait`, and that definition must occur either in the crate defining the type or the crate defining the `trait`.

### Match arms

The arms of `match` (previously `alt`) expressions have a new syntax. Each case is now followed by a fat arrow, `=>`, and the block is optional. Cases are separated by commas.

```
// The new syntax is quite compact for one-line cases
match foo {
  bar => baz,
  quux => fail
}

match foo {
  bar => {
    baz
  } // Note: no comma needed when using braces
  quux => {
    fail
  }
}
```

See also [#3057](https://github.com/mozilla/rust/issues/3057).

### Static methods and explicit self types

static methods
explicit self types

### Transitioning import/export to pub/priv

([#2300](https://github.com/mozilla/rust/issues/2300)) There was some tension in readability between "ease of scanning a module's exports" and "ease of reading the code and knowing which item is exported when you're looking at it." Ultimately we came down on the side of maintainability: that it's less work for a maintainer to mark the items where they occur, rather than scrolling back and forth between export-list and item definitions. Along with moving exports to the items themselves, we changed the terms to use the same keywords used for access control in structs: `pub` and `priv`.

### Change `use` to mean `import`, remove `import`, make crate-linkage use `extern mod foo = (...);`

(also [#2082](https://github.com/mozilla/rust/issues/2082)) Many Rust programmers stub their toes on the difference between `import` and `use`; both "read like" they should somehow make-available the elements in the target module. Since removing `export` (see above) there is an asymmetry in the keywords anyways, so we removed the keyword `import`, switched `use` to mean what `import` previously meant, and now denote crate-linkage through `extern mod foo = (...)`.

### Separate form of module import: `use mod foo = bar;`

(also [#2082](https://github.com/mozilla/rust/issues/2082)) This has to do with making the resolve pass coherent. In the old resolve code, resolving modules and resolving items glob-imported _from_ modules was intermixed, and could lead to incoherence of the algorithm. In the new resolve code (landed in 0.3) there is a separation of passes: module-imports are not run through globs, only module-to-module renamings, and are resolved _first_, and then all imports through modules (including glob-imports) are resolved _after_. The module-import syntax is therefore separate, written as `use mod foo = bar;` to reflect this algorithmic separation.

### Macro changes

Macros are now invoked with a postfix `!` instead of prefix `#`, as in `debug!("foo")`.

We also have a powerful new way to define macros with the `macro_rules!` syntax extension. Macros are now based on trees of tokens with balanced braces instead of the full AST expressions the old macro implementation used.

See also [the macro tutorial][macros].

[macros]: http://dl.rust-lang.org/doc/tutorial-macros.html

### Smaller syntax changes

* alt -> match
* use -> extern mod
* Region names are specified as `&r/T` instead of `&r.T`

### Kinds become traits

### Deprecation / removal of argument modes

([#2030](https://github.com/mozilla/rust/issues/2030))  Previous versions of Rust attempted to select optimal default argument-passing behavior (by-reference or by-value) based on type-directed heuristics, with sigils available to override the defaults. This system (called "modes" or "argument modes") was responsible for the presence of the sigils `+`, `-`, `&` and `&&` on function parameters.

In practice this system was simultaneously too confusing to use and too inflexible with respect to scenarios where first-class borrowed pointers were desired (returning by reference, storing non-escaping pointers into structured values, etc.) Therefore in 0.4, as first-class borrowed pointers (the other, more pervasive use of the syntax `&T`) have matured to be a better alternative in almost all cases, and many of the residual motives for modes made redundant by full monomorphization, we have deprecated (and removed in as much code as possible) the mode system; it should not be visible to users and support for it is only available by opting in with the `#[legace_modes]` crate attribute.

### Inherited mutability

([nmatsakis:mut](http://smallcultfollowing.com/babysteps/blog/2012/05/31/mutability/)) After several repeated attempts to treat mutability as a full type constructor, we have settled on treatment as an _inherited_ storage-qualifier. That is, while the `mut` keyword can still only be used to qualify storage locations (the referents of pointers, variables, fields in structures), by placing `mut` on a storage location you now implicitly deeply `mut`-qualify all of the owned (sendable) types within the mutable location. This makes sense if you consider that an "outer" `mut` qualification could always _effectively_ mutate an inner "immutable" qualification by simply overwriting the entire containing location. This change therefore only makes the rule explicit in the type system; it is in other words a matter of improving the type system's soundness with respect to mutability.

```
struct S1 {
  field: ~S2;
}

struct S2 {
  intfield: int,
  boxfield: @int
}

let mut = S1 { field: S2 { intfield: 1, boxfield: @2 } };

// S1 is in a mutable slot so we can mutate the fields
S1.field = S2 { intfield: 3, boxfield: @4 };

// We can also reach through owned boxes and mutate their insides
S1.field.intfield = 5;

// Managed boxes are not owned so their interior is not mutable
*S1.field.boxfield = 6; // ERROR
```

At first glance this seems like an odd way to deal with mutability, but it seems to work very well with Rust's ownership semantics, and it enables some fascinating patterns that are particularly useful for concurrency. In particular it allows for dual mode data structures that, under certain circumstances my mutate fields, and under others not, similar to C++ const methods.

```
struct S {
  foo: int
}

impl S {
  // Method declared with explicit mutable self type
  fn mutate(&mut self) {
    self.int = 10;
  }

  fn do_not_mutate(&self) {
    // Can't mutate fields because we don't have a mutable self pointer
  }
}
```

So you can do cool things with this.

```
// LinearMap is an owned map type in core::send_map
// While it lives in a mutable slot we can call methods that mutate the map
let mut map = LinearMap();
map.insert(foo, bar);

// Move it into an immutable slot and the whole structure becomes 'frozen'
let map = move map;
map.insert(foo, bar); // ERROR

// At this point you can do some interesting things like put your entire map
// into a read-only shared-memory container (ARC - atomically reference counted).
let arc_map = ARC(move map);

for repeat(50) {
  // Create another handle to the same map
  let arc_clone = arc::clone(arc_map);
  do spawn |move arc_clone| {
    // Do cool things with my immutable shared-memory hash map
  }
}
```

### Pipes

## 0.3 July 2012

### Integer-literal suffix inference

Rust has no coercion between integral types, so until this release
literals always required an appropriate suffix for types other than
`int`, e.g. `12_u8`. This was widely considered an eye-sore and
inconvenience.  Now these suffixes can be left off in most situations
and their type will be correctly inferred.

    let random_digits: [u8] = [1, 4, 1, 5, 9, 2, 6];

Read the detailed [announcement on the mailing list][inference].

[inference]: https://mail.mozilla.org/pipermail/rust-dev/2012-July/002002.html

### Class stabilization, removal of resources

Classes are ready for use now, and can contain destructors and implement ifaces.
The resource type has been removed in favor of class destructors. In addition, both classes and class methods can have type parameters.

    iface talky {
        fn speak();
    }

    iface printable {
        fn print();
    }

    class talker<T: printable> : talky {

        let word: T;

        new(word: T) {
            self.word = word;
        }

        drop {
            self.speak();
        }

        fn speak() {
            self.word.print();
        }
    }

### New closure syntax, new `do` syntax for control-structure-like calls

Closures have a more compact syntax: `foo.map( |i| i + 1)`

They work with the overhauled `for` loops and the new `do` expressions to
provide nice sugar for higher-order functions.

    for foo.each |i| {
        println(#fmt("%?", i));
    }

With no arguments these forms are quite minimal.

    do spawn {
        // this is the body of a closure
    }

Read [full discussion][closures].

[closures]: https://mail.mozilla.org/pipermail/rust-dev/2012-July/002000.html

### New vector types

Until now all vectors have been unique types allocated on the exchange
heap, but accidental copying of and allocation of vectors turned out
to be a major performance hazard.

0.3 features the full complement of vector types demanded by Rust's
memory model - in other words you can put vectors on the stack, the
local heap or the exchange heap.

    // A unique vector, allocated on the exchange heap
    let x: ~[int] = ~[0];
    // A shared vector, allocated on the local heap
    let y: @[int] = @[0];
    // A stack vector, allocated on the stack
    let z: &[int] = &[0];

The libraries have not fully caught up to the new vector types.

> Note: `[]` is currently a synonym for `~[]`. It's not clear what the former syntax will end up meaning,
> possibly a slice or fixed-length vector.

Some posts on vectors that have varying relationships to the final implementation:

* [Arrays, vectors](https://mail.mozilla.org/pipermail/rust-dev/2012-March/001467.html)
* [Arrays, vectors, redux](https://mail.mozilla.org/pipermail/rust-dev/2012-March/001476.html)
* [A Vision for Vectors](https://mail.mozilla.org/pipermail/rust-dev/2012-June/001951.html)
* [Vectors, Slices and Functions, Oh My](http://smallcultfollowing.com/babysteps/blog/2012/05/14/vectors/)

### *-patterns

We have a new syntax for ignoring variant fields in patterns

    alt my_enum {
        i_could_match_like_this(_, _, _, _, _, _) {
        }
        but_would_rather_like_this(*) {
        }
    }

### Doc comments

Rust now has a form of attribute specifically for doc comments. Like other
attributes there are different forms depending on whether the documentation
is on the outside or the inside of the thing its documenting.

    /** Outer doc comment */
    fn f() {

    /// Outer doc comment
    fn g() { }

    fn h() {
        /*! Inner doc comment */
    }

    fn i() {
        //! Inner doc comment
    }

### Per item control over warnings and errors

Warnings can be disabled (or enabled or elevated to errors)
on a per-item basis, whereas before it was per-crate.

    #[warn(no_non_implicitly_copyable_typarams)]
    fn implicitly() -> ~[str] {
        import std::sort::merge_sort;

        merge_sort(str::eq, ~["not_implicitly_copyable"])
    }

In this example, `merge_sort` will copy the elements of the vector
during sorting, but strings may not be copied without writing `copy`
(because they are unique types and copying them involves
allocation). In this situation rustc currently emits a warning. As
there is no language-level mechanism yet to authorize the copy,
sometimes you just have to turn the warning off.

Note that the current syntax for disabling warnings by prefixing "no_"
to the naming of the warning is confusing and will change.

The warnings (and their default setting) understood by rustc are:

* ctypes (warn) - foreign modules should use core::libc types
* unused_imports (ignore) - disallow imports that are never used
* while_true (warn) - disallow `while true` (should use loop)
* path_statement (warn) - writing a statement that names a value without
                          using it is usually a bug
* old_vecs (warn) - use of deprecated vec syntax (`[]`, the meaning of
                    which is likely changing to slice instead of unique vector.
                    the current syntax for unique vectors is `~[]`)
* old_strs (ignore) - use of deprecated str syntax
* unrecognized_warning

The following are all related to the in-progress effort to eliminate expensive,
implicit copies from the language. The current rule is that types that require
allocation to copy (unique boxes) or that contain mutable fields
are not implicitly copyable. These are warnings now but will be either errors
by default or entirely disallowed by the typesystem in the future.

* implicit_copies - performing a copy of a non-implicitly copyable type without
                   the `copy` keyword
* vecs_not_implicitly_copyable - same as above but for vectors
* non_implicitly_copyable_typarams - use of generics whose type parameters
                                     have the `copy` kind with a type that
				     is not implicitly copyable

### Region progress

There has been a lot of progress in converting Rust to a region-based memory model.
Although region pointers are not yet ready for use, internally many of rust's
analyses have been rewritten in terms of regions.

See Niko's blog posts about regions:

* [References](http://smallcultfollowing.com/babysteps/blog/2012/04/25/references/)
* [Borrowing](http://smallcultfollowing.com/babysteps/blog/2012/05/01/borrowing/)
* [Borrowing errors](http://smallcultfollowing.com/babysteps/blog/2012/05/05/borrowing-errors/)

### New syntax extensions

    import io::println;

    // Typical file info
    println(#fmt("%?", #line()));
    println(#fmt("%?", #col()));
    println(#fmt("%?", #file()));

    // The name of the current module, or empty
    println(#fmt("%?", #mod()));

    let x = 10, y = 15;

    // Turn a Rust expression into a string
    println(#fmt("%?", #stringify[x + y]));
    // Include the contents of a file as a Rust expression
    println(#fmt("%?", #include("x_plus_y.rs")));
    // Include the contents of a file as a string
    println(#fmt("%?", #include_str("x_plus_y.rs")));
    // Include the contents of a file as a byte vector
    println(#fmt("%?", #include_bin("x_plus_y.rs")));

### Const kind

Rust has a new kind, `const`, that can be used as a bounds on type
parameters. A type that is const contains only const fields that are
not mutable. A type that is both `const` and `send` is appropriate for
using in shared-memory concurrency patterns because it is deeply
immutable and does not contain local box pointers.

This is currently used by `core::arc` which provides an atomically
reference counted, sendable type that encapsulates a `const` + `send`
type.

### FFI changes

The word 'native' is being expunged from the language since it implies
that Rust is not native. Native modules are now declared with the
`extern` keyword.

    extern mod cairo {

Rust functions that can be called from native code with the CDECL ABI,
previously called 'crust', are now also declared with the `extern'
keyword.

    extern fn a_native_callback(user_data: *c_void) {
         do_some_crusty_stuff(user_data);
    }

These changes are part of a [larger plan][crust] to overhaul the terminology
and syntax around linking.

[crust]: https://github.com/mozilla/rust/issues/2082#issuecomment-6521683

### Shebang

The first line of a Rust source file can contain a shebang

    #! /usr/local/bin/rustx

### Removed features

`be`, `prove`, `syntax`, `note` were unimplemented and removed from
the language.  `mutable` is now written `mut`. `cont` was renamed
to `again`. `bind` had too much overlap with other closure forms 
while providing subtly different semantics so was removed. `do` 
loops were rarely used, so the `do` keyword was repurposed. Resources
were removed in favor of class destructors.

### Reflection system

A preliminary reflection system now exists. Type descriptors contain 
a compiler-generated function that calls visitor-methods on a predefined
intrinsic visitor interface. This enables reflecting on a value without
knowing its type (with some supporting library work). Much existing code
will gradually shift over to this interface, as it subsumes a number of
other tasks the compiler and runtime are currently doing as special cases.

### Library improvements

There are more methods available on more basic and core types by
default now. Methods are generally preferred over functions
now when there is a clear 'self' type.

The standard library has a new `time` module.

    let time: tm = now();

    // Convert to a string
    println(time.strftime("%D/%M/%Y"));

    // tm has some built in conversions
    println(time.ctime());   // "Thu Jan  1 00:00:00 1970"
    println(time.rfc822());  // "Thu Jan  1 00:00:00 1970"
    println(time.rfc3339()); // "2012-02-22T07:53:18-07:00"

### TODO

Need to write about:

* UV-related APIs

## 0.2 March 2012

### Region pointers

These are a new kind of pointer. In this release, they are written with the sigil `&`, which is slightly ambiguous with the argument-mode sigil `&` but our longer-term plan is to replace all argument modes with region pointers and remove the concept of argument modes. So this ambiguity was tolerated during this release cycle (we may pick a different sigil later).

Region pointers are cheaper to use than any other sorts of safe pointer in rust; they are statically guaranteed to point to live memory (by construction) so anywhere you are allowed to use them, it is safe to use them, and manipulating them incurs no cost -- not even garbage-collection scanning cost. They are much like C++ `&`-references, they way they are idiomatically used. We recommend all signatures to functions be written in terms of region pointers whenever possible. The rust compiler will eventually support automatically "borrowing" the other sorts of pointers into region pointers for the duration of a call, such that region pointers can act as a suitable default for callee signatures.

### Operating system and libc interfaces

The previous interface was written in terms of a number of per-platform modules in `std`, each of which exposed a sub-module called `libc`, as well as some redundant interfaces in `std::fs` and `std::os_fs`. There was a lot of redundant and poorly-factored, platform-variable code in here. It was consolidated into three files, `core::libc`, `core::os`, and `core::fs` which were platform-conditionalized on an item-by-item basis.

## 0.1 January 2012

Initial release.