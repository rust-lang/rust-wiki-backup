This page covers releases in more detail than the bullet-point list given in the RELEASES.txt file in the source distribution, in particular focusing on _language level changes_ that will be immediately visible and/or disruptive to users trying to keep their Rust code compiling and working right between releases. It is intended to hold copied, cleaned-up versions of entries from [[Note development roadmap]] as they are completed, to help users plan migration on their own code.

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

* implicit_copys - performing a copy of a non-implicitly copyable type without
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

    fn manually_share_arc() {
        // unique vectors are const and send kinds
        let v = ~[1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
        let arc_v = arc::arc(v);

        let p = port();
        let c = chan(p);

        do task::spawn() {
            let p = port();
            c.send(chan(p));

            let arc_v = p.recv();

            let v = *arc::get::<~[int]>(&arc_v);
            assert v[3] == 4;
        };

        let c = p.recv();
        c.send(arc::clone(&arc_v));

        assert (*arc::get(&arc_v))[2] == 3;

        log(info, arc_v);
    }


### Shebang

The first line of a Rust source file can contain a shebang

    #! /usr/local/bin/rustx

### Removed features

`be`, `prove`, `syntax`, `note` were unimplemented and removed from
the language.  `mutable` is now written `mut`. `bind` had too much
overlap with other closure forms while providing subtly different
semantics so was removed. `do` loops were rarely used, so the `do`
keyword was repurposed. Resources were removed in favor of class
destructors.

### TODO

Need to write about:

* eliminating copies
* FFI changes
* Time functions
* UV-related APIs
* Const kind

## 0.2 March 2012

### Region pointers

These are a new kind of pointer. In this release, they are written with the sigil `&`, which is slightly ambiguous with the argument-mode sigil `&` but our longer-term plan is to replace all argument modes with region pointers and remove the concept of argument modes. So this ambiguity was tolerated during this release cycle (we may pick a different sigil later).

Region pointers are cheaper to use than any other sorts of safe pointer in rust; they are statically guaranteed to point to live memory (by construction) so anywhere you are allowed to use them, it is safe to use them, and manipulating them incurs no cost -- not even garbage-collection scanning cost. They are much like C++ `&`-references, they way they are idiomatically used. We recommend all signatures to functions be written in terms of region pointers whenever possible. The rust compiler will eventually support automatically "borrowing" the other sorts of pointers into region pointers for the duration of a call, such that region pointers can act as a suitable default for callee signatures.

### Operating system and libc interfaces

The previous interface was written in terms of a number of per-platform modules in `std`, each of which exposed a sub-module called `libc`, as well as some redundant interfaces in `std::fs` and `std::os_fs`. There was a lot of redundant and poorly-factored, platform-variable code in here. It was consolidated into three files, `core::libc`, `core::os`, and `core::fs` which were platform-conditionalized on an item-by-item basis.

## 0.1 January 2012

Initial release.