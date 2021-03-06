This page covers releases in more detail than the bullet-point list given in the RELEASES.txt file in the source distribution, in particular focusing on _language level changes_ that will be immediately visible and/or disruptive to users trying to keep their Rust code compiling and working right between releases.

*Note: As of 1.0.0-alpha (January 2015) detailed release notes are no longer produced. Instead the short release notes are extensively cross-referenced with supporting RFC's and other documentation.*

## 0.12.0 October 2014

This development cycle saw a renewed push to get the final set of 1.0 language features in order, but also a massive improvement in documentation, in Windows support, and library stabilization. This was also the development cycle in which [Cargo](http://crates.io) matured significantly, receiving widespread adoption.

### Lifetime elision

Lifetime annotations can be left off of function definitions in several common cases.

#### Input and Output lifetimes

Lifetime positions can appear as either "input" or "output":

* For `fn` definitions, input refers to the types of the formal arguments
  in the `fn` definition, while output refers to
  result types. So `fn foo(s: &str) -> (&str, &str)` has elided one lifetime in
  input position and two lifetimes in output position.
  Note that the input positions of a `fn` method definition do not
  include the lifetimes that occur in the method's `impl` header
  (nor lifetimes that occur in the trait header, for a default method).


* For `impl` headers, input refers to the lifetimes appears in the type
  receiving the `impl`, while output refers to the trait, if any. So `impl<'a>
  Foo<'a>` has `'a` in input position, while `impl<'a, 'b, 'c>
  SomeTrait<'b, 'c> for Foo<'a, 'c>` has `'a` in input position, `'b`
  in output position, and `'c` in both input and output positions.

#### The rules

* Each elided lifetime in input position becomes a distinct lifetime
  parameter. This is the current behavior for `fn` definitions.

* If there is exactly one input lifetime position (elided or not), that lifetime
  is assigned to _all_ elided output lifetimes.

* If there are multiple input lifetime positions, but one of them is `&self` or
  `&mut self`, the lifetime of `self` is assigned to _all_ elided output lifetimes.

* Otherwise, it is an error to elide an output lifetime.


#### Examples

```rust
fn print(s: &str);                                      // elided
fn print<'a>(s: &'a str);                               // expanded

fn debug(lvl: uint, s: &str);                           // elided
fn debug<'a>(lvl: uint, s: &'a str);                    // expanded

fn substr(s: &str, until: uint) -> &str;                // elided
fn substr<'a>(s: &'a str, until: uint) -> &'a str;      // expanded

fn get_str() -> &str;                                   // ILLEGAL

fn frob(s: &str, t: &str) -> &str;                      // ILLEGAL

fn get_mut(&mut self) -> &mut T;                        // elided
fn get_mut<'a>(&'a mut self) -> &'a mut T;              // expanded

fn args<T:ToCStr>(&mut self, args: &[T]) -> &mut Command                  // elided
fn args<'a, 'b, T:ToCStr>(&'a mut self, args: &'b [T]) -> &'a mut Command // expanded

fn new(buf: &mut [u8]) -> BufWriter;                    // elided
fn new<'a>(buf: &'a mut [u8]) -> BufWriter<'a>          // expanded

impl Reader for BufReader { ... }                       // elided
impl<'a> Reader for BufReader<'a> { .. }                // expanded

impl Reader for (&str, &str) { ... }                    // elided
impl<'a, 'b> Reader for (&'a str, &'b str) { ... }      // expanded

impl StrSlice for &str { ... }                          // elided
impl<'a> StrSlice<'a> for &'a str { ... }               // expanded

// elided
trait Bar<'a> {
  fn bound(&'a self) -> &int { ... }
  fn fresh(&self) -> &int { ... }
} 

// expanded
trait Bar<'a> {
  fn bound(&'a self) -> &'a int { ... }
  fn fresh<'b>(&'b self) -> &'b int { ... }
}

// elided
impl<'a> Bar<'a> for &'a str {
  fn bound(&'a self) -> &'a int { ... }
  fn fresh(&self) -> &int { ... }
}

// expanded
impl<'a> Bar<'a> for &'a str {
  fn bound(&'a self) -> &'a int { ... }
  fn fresh<'b>(&'b self) -> &'b int { ... }
}
```

See also the [RFC](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0039-lifetime-elision.md).

### Where clauses

The current syntax for placing trait and region bounds on type parameters, `<T: Foo + Bar + 'baz>`, is not sufficient
for some future use-cases, including [associated types](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0059-associated-items.md).
Furthermore, with many bounds, the syntax can become difficult to read and obscure more important aspects of declarations.
`where` clauses are a new and more flexible syntax for specifying bounds, which come *after* the rest of the declaration.

```rust
trait Equal {
    fn equal(&self, other: &Self) -> bool;
    fn equals<T,U>(&self, this: &T, that: &T, x: &U, y: &U) -> bool
            where T: Eq, U: Eq;
}

impl<T> Equal for T where T: Eq {
    fn equal(&self, other: &T) -> bool {
        self == other
    }
    fn equals<U,X>(&self, this: &U, other: &U, x: &X, y: &X) -> bool
            where U: Eq, X: Eq {
        this == other && x == y
    }
}

fn equal<T>(x: &T, y: &T) -> bool where T: Eq {
    x == y
}

fn main() {
    println!("{}", equal(&1i, &2i));
    println!("{}", equal(&1i, &1i));
    println!("{}", "hello".equal(&"hello"));
    println!("{}", "hello".equals::<int,&str>(&1i, &1i, &"foo", &"bar"));
}
```

It has not been decided whether the old syntax will be removed.

See also the [RFC](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0066-where.md).

### Slice notation

Slice notation allows for conveniently taking a view over a collection. Examples:

```
let v =               vec!['a', 'b', 'c', 'd', 'e', 'f'];

let slice1 = v[1..4]; //      &['b', 'c', 'd']
let slice2 = v[..4];  //      &['a', 'b', 'c', 'd']
let slice3 = v[1..];  //      &['b', 'c', 'd', 'e', 'f']
```

It can also be used to conveniently convert `Vec<T>` to `&[T]` or `String` to `&str`:

```
let a: Vec<int> = vec![0, 1];
let b: &[int] = a[];

let c: String = "foo".into_string();
let d: &str = c[];
```

Slicing is an overloadable operator, implemented via the `Slice` and `SliceMut` traits.

See also the [RFC](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0058-slice-notation.md)

### Dynamically-sized types

The work on dynamically sized types allows types which don't have a statically known size to be used much like other types in Rust. Previously, types such as arrays without a fixed length and trait types could not be used in many places and had special treatment in the compiler. Now these types can have `impl`s and can be used as type parameters. That means you can write implement a trait for arrays without caring how the array is referenced (e.g., `impl<T> Foo for [T] { ... }`) and use your own smart pointers with trait objects and arrays (e.g., `Rc<[int]>` or `Rc<Show>`).

### Implemented RFC's

Decisions about what features to add to Rust are driven by an [RFC (request for comments) process](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/README.md). This is a partial list of the RFC's that impacted the 0.12.0 development cycle.

* [Lifetime elision](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0039-lifetime-elision.md)
* [Bounds on objects and generic types](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0049-bounds-on-object-and-generic-types.md)
* [Unifying (unboxed) closures and traits](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0044-closures.md)
* [Capture inference for (unboxed) closures](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0063-upvar-capture-inference.md)
* [`if let`](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0051-if-let.md)
* [`while let`](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0068-while-let.md)
* [Tuple indexing](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0053-tuple-accessors.md)
* [Feature gate advanced slice patterns](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0054-feature-gate-slice-pats.md)
* [Subslice syntax](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0055-subslice-syntax-change.md)
* [Add variants to type namespace](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0061-variants-namespace.md)
* [More expressive `cfg` syntax](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0065-cfg-syntax.md)
* [Remove `Gc<T>`](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0067-remove-refcounting-gc-of-t.md)
* [Rename `Share`](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0045-share-to-threadsafe.md)
* [No shadowing of view items](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0050-no-module-shadowing.md)
* [Less leaking of private items](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0048-no-privates-in-public.md)
* [Change the renaming `use` syntax](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0047-use-path-as-id.md)
* [Remove special borrow checking for `Box<T>`](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0043-box-not-special.md)
* [Remove `crate_id` attribute](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0035-remove-crate-id.md)
* [Index traits](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0034-index-traits.md)
* [Remove 'cross-borrowing'](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0033-remove-cross-borrowing.md)
* [Remove 'cross-borrowing' redux](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/complete/0037-remove-cross-borrowing-entirely.md)
* [Remove runtime/IO abstraction](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0062-remove-runtime.md)
* [`where` clauses](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0066-where.md)
* [Slice notation](https://github.com/rust-lang/rfcs/blob/d2c2f0f524df814d7b38f69311ab67f41c2ec3ec/active/0058-slice-notation.md)

## 0.11.0 July 2014

While this was a very active development cycle, it was largely focused on polishing the type system and libraries. The major technical focuses this time were implementing [dynamically sized types](http://smallcultfollowing.com/babysteps/blog/2014/01/05/dst-take-5/) and [refactoring the standard library](https://github.com/rust-lang/rfcs/blob/master/text/0040-libstd-facade.md). This release also marks the complete removal of the `~` and `@` syntax in favor of library types `Box` and `Gc` (though there is still work to be done with the library integration: the compiler still has more knowledge of `Box` and `Gc` than it ultimately should, but the final syntax is in place).

Additionally, the Rust [RFC process](https://github.com/rust-lang/rfcs/blob/master/text/0002-rfc-process.md) continued to mature, and new measures were put into place to make forward porting Rust code to new versions easier, including a [breaking changes log](https://mail.mozilla.org/pipermail/rust-dev/2014-April/009543.html) and increase use of library [deprecation](http://doc.rust-lang.org/rust.html#stability) when making API changes.

### DST, vectors, and strings

In preparation for an implementation of dynamically-sized-types (DST), the `~str` and `~[T]` types have been removed from the language. These will be reincarnated as `Box<str>` and `Box<[T]>` (`Box` explained below), but currently do not exist today. These types are replaced by `String` and `Vec`.

#### `Vec`

The `Vec` type replaces the old `~[T]` type and brings with it some significant speed improvements. The layout of `Vec` has been altered from that of `~[T]` to take much greater advantage of LLVM's optimizations passes.

Like `~[T]`, `Vec<T>` is a growable vector which can be done through a mutable pointer. It is freely borrowable to `&mut [T]` and `&[T]`.

#### `String`

This is essentially a drop-in replacement for the `~str` type. It is the default "owned string" type of Rust, and supports mutation through a mutable pointer. It is backed by a `Vec<u8>` type and has the runtime invariant that it only ever carries bytes which are a valid utf8 encoding. The type is freely borrowable to the `&str` type via the `as_slice()` method.

The `String` type is in the prelude and lives in `std::string`.

### Removal of `~` and `@`

The ubiquitous `~` and `@` pointers have been removed from the language to be relegated to libraries. The type syntax `~T` is replaced by `Box<T>` which lives in `std::owned`, and the type syntax `@T` was replaced by `Gc<T>` which lives in `std::gc`.

The expression syntax, `~expr`, has been replaced by `box expr` and the `@expr` syntax has been replaced by `box(GC) expr` where `GC` lives in `std::gc`. The `box` expressions is Rust's equivalent of "placement-new" and will support user-defined allocators in the future, but it currently only supports `Box` and `Gc` pointers.

This is largely just a syntactic change, but it is only one milestone in the long trail of features being moved from the compiler into libraries instead. With this change comes simplified compiler support, improved discoverabilty in documentation, and a more consistent standing next to other smart pointers such as `Rc` and `Arc`.

The major significance of this change is spurred by DST after which `~` and `@` will no longer need to be specially treated and hence have no need for being specially recognized by the compiler.

### The "`std`" facade

The standard library, `libstd`, has been refactored to instead be an umbrella library encompassing a number of smaller libraries underneath it. The purpose of this refactoring was to have clear lines of abstraction between libraries, as well as better understanding the dependencies of each component library. The new libraries introduced under the standard library are:

* `libcore` - This library is intended to be the core functionality for all of Rust. It is a "0-assumption" library which is theoretically maximally portable (no dependencies). The library does not depend on a system libc, for example. This library is suitable for use in contexts such as embedded devices, kernel development, or a Rust library embedded into another language. 
  
  The libcore library does not know how to allocate memory, so constructs such as growable vectors and growable strings are not present. The library does rely, however, on an upstream definition of failure. The only requirement for this failure function is that it never return, however. More information about this library can be found in its [documentation](http://doc.rust-lang.org/core/)

* `liblibc` - This library is used to represent bindings to libc on the relevant platform desired. It has appropriate linkage directives to link to necessary system libraries, as well as all the necessary definitions for calling functions.

* `liballoc` - This library is the core allocation interface of Rust. Core "smart pointers" such as `Rc<T>`, `Arc<T>`, and `Box<T>` are provided by this library. Additionally, all memory allocation in Rust is now performed through this library. Currently jemalloc is used to power `liballoc` by default. This library depends on `liblibc` and `libcore`.

* `libcollections` - This library provides the core collection types of the language, such as vectors, strings, maps, linked lists, etc. This library only depends on `liballoc` and `libcore`, in theory not even depending on libc.

* `librustrt` - This library is the core component of the runtime, defining the I/O interfaces, task model, and unwinding. This library provides the definition of failure for all upstream rust crates as unwinding via the system libunwind.

* `libsync` - One of the last libraries before you get to the standard library, this library was refactored from depending on the standard library to having the standard library depend on it. The libsync library is responsible for providing the primitives necessary for writing concurrent code in Rust such as channels, mutexes, etc.

* `librand` - As with libsync, this library was refactored from depending on the standard library to having the standard library depend on it. As a consequence, however, this library no longer defines primitives such as `OsRng`. Instead, the librand library only depends on libcore, and the missing functionality is all provided by the standard library.

None of these crates should normally be linked to explicitly. Instead, the standard library reexports all necessary functionality required. For example, the following modules exist in the standard library now:

* `std::collections`
* `std::sync`
* `std::comm`
* `std::rand`
* `std::rt`

And various other explicitly reexported modules such as all of core, `rc`, `owned`, etc.

With this facade of the standard library, it should be easier to pick and choose the amount of functionality from Rust. It's much easier to write idiomatic rust without the runtime now, for example, through usage of just the `libcollections` crate.

## 0.10 April 2014

### Removal of `libextra`

The conglomerate crate `libextra` has been removed from the standard distribution. Its component modules have been split out into other crates as seen appropriate. In the future, these crates will live on a cargo server rather than in the main repository, but currently there is no other place for them to reliably reside.

The full list of component crates is currently:

* libarena
* libcollections
* libflate
* libgetopts
* libglob
* libnum
* libsemver
* libserialize
* libsync
* libterm
* libtest
* libtime
* libuuid
* liburl
* libworkcache

In addition to this splitting of libextra, functionality has been split outside of `libstd`:

* librand
* liblog

This effort to break up libraries into their components is part of an ongoing effort to distill libraries to their core set of dependencies in order to be usable in as many contexts as possible. As an added bonus, this also helps parallelize builds quite a bit when so many libraries can be built in parallel.

As with the rest of Rust's current libraries, these names and interfaces are not yet stable. They are still likely to change over time as more stable apis evolve and emerge. Also note that while all of these crates are currently part of the standard distribution, that will likely not be the case in the future as they migrate to their own cargo packages outside of the Rust repository.

### Removal of conditions, effect on I/O

During this release cycle, the `std::conditions` module was removed. Details and rationale as to why can be found in the [relevant commit](https://github.com/mozilla/rust/commit/454882dcb7fdb03867d695a88335e2d2c8f7561a).

This removal implies that all I/O no longer raise a condition when an error happens. Instead, all I/O operations now return a value of the `IoResult` type. This is a thin typedef: `type IoResult<T> = Result<T, IoError>;`. This change should enable handling I/O errors much more natural to write than watching for conditions being raised.

One of the main benefits of conditions was the guarantee that no error would go unnoticed. In order to not entirely give up this benefit, two strategies are employed in the compiler to ensure that errors are handled:

1. A new `unused_must_use` lint was introduced when will emit a warning whenever a type flagged with `#[must_use]` is not used. The formal definition is that any semicolon-delimited statement whose type is not `()` will have a warning emitted if the type is flagged as `#[must_use]`. This check will catch errors such as:

    ```rust
    let mut file = File::open(path);
    file.write([10]); //~ ERROR: unused result
    ```

2. A new macro, `try!` is provided to short-circuit functions dealing with lots of I/O errors. Example usage looks like:

    ```rust  
    // try!(e) => match e { Ok(e) => e, Err(e) => return Err(e) }

    fn has_valid_data(file: &mut File) -> io::IoResult<bool> {
        let num_bytes = try!(file.read_be_u32());
        let data = try!(file.read_exact(num_bytes));
        let cksum_len = try!(file.read_be_u16());
        let cksum = try!(file.read_exact(cksum_len));

        Ok(checksum_matches(data, cksum))
    }
    ```

These two tools provide the benefits of conditions while results provide a much more natural syntax for dealing with errors.

### Cross-crate syntax extensions

Macros can be defined with `macro_rules!` in one crate and used in another. A new attribute, `#[macro_export]` (not to be confused with `#[macro_escape]`) has been added. A macro definition tagged with this attribute will be made available to code linking against the crate it's defined in.

```rust
#[macro_export]
macro_rules! foo (
    () => (println!("foo"))
)
```

Crates that wish to use `foo!` should annotate the `extern crate` statement with another new attribute, `#[phase(syntax)]` to tell the compiler to load macro definitions from it:

```rust
#[phase(syntax)]
extern crate my_crate;
...
foo!();
```

The `#[phase(syntax)]` attribute will cause the crate to be used *only* for macros. If you want to use the crate as you normally would, you can add the `link` argument to the attribute as well:

```rust
#[phase(syntax, link)]
extern crate my_crate;
...
foo!();
my_crate::bar();
```

An `extern crate` statement that is un-annotated is equivalent to one annotated with `#[phase(link)]`.

In addition, procedural macros, such as `bytes!`, `env!`, and `println!` can now be defined in third-party crates outside of the compiler and imported into crates using the same `#[phase(..)]` syntax discussed above. The procedural macro API is currently tied closely to `rustc`'s internals and is therefore highly unstable. For more information on procedural macros, see the [relavent pull request](https://github.com/mozilla/rust/pull/11151) or the new [libfourcc](https://github.com/mozilla/rust/blob/b06b3667afb3ca85f118ca6f91dcc592afae4a42/src/libfourcc/lib.rs) and [libhexfloat](https://github.com/mozilla/rust/blob/b06b3667afb3ca85f118ca6f91dcc592afae4a42/src/libhexfloat/lib.rs) crates.

### Changes to temporary lifetimes

Temporaries are frequently created as part of intermediate expressions in order to calculate a final result. A good example of this is the `borrow()` method on `RefCell`. This method returns a new temporary `Ref` which can be dereferenced to the contained data.

Before 0.10, the lifetimes of temporaries were much shorter than often required, forbidding common code. The compiler now generates code where temporaries live to the "innermost enclosing statement." Essentially, code like this can now work on 0.10:

```rust
let slot = RefCell::new(1);
*slot.borrow_mut() = 4;
println!("{}", *slot.borrow());
```

In this example, the `slot.borrow_mut()` method returns a value of type `RefMut`, but it is a temporary because it is never named. This temporary is deallocate at the end of the innermost enclosing statement, which in this case is `*slot.borrow_mut() = 4;`. This causes the `RefCell` to not be borrowed by the time the second statement comes around.

You'll likely notice that many temporary expressions which previously must have been lifted into a local will no longer need the special treatment.

For the full details of this change, see #3511 and #11585.

### The `Deref` and `DerefMut` traits

These two new lang item traits were added as the key ingredient to Rust's smart pointer story. These are essentially overloads of the `*` operator. For example:

```rust
struct Foo { a: int }
impl Deref<int> for Foo {
    fn deref<'a>(&'a self) -> &'a int { &self.a }
}

let test = Foo { a: 3 };
println!("{}", *test); // prints "3"
let borrowed: &int = &*test; // dereference, but don't move
println!("{}", *borrowed);
```

The `DerefMut` trait allows overriding expressions of the form `*test = value;`, as well as allowing taking a mutable address to the contents as `&mut *test`.

These two traits participate in the compiler's autoderef functionality, making "smart pointers" much easier to use. For example, `Rc<T>` now "implements the methods of T" by auto-dereferencing to `&T`. Given a value of `Rc<T>`, you can seamlessly invoke methods on the contained object.

### Changes in treatment of `static` variables

Values in `static` have seen some treatment as part of 0.10. The changes happened incrementally over the release, but the current semantics are described by:

* `static` expressions must not be of a type that has a user-defined destructor
* `static` expressions must not have a value which itself requires a destructor
* `static` expressions which contain an `Unsafe<T>` cannot have their address taken
* `static mut` expressions must not have a type which requires any destructor at all

These rules should allow for non-copyable types in statics such as `AtomicUint` as well as `Option<~int>`, while disallowing types in mutable statics that need destructors because there is no time at which the destructors will be run.

## 0.9 January 2014

### The deprecation of `@`, its alternatives, and interior mutability

As part of the final sequence of major language changes, Rust is being overhauled to have fewer baked-in pointer types, and be more extensible to custom pointer types (smart pointers). In this release what have previously been termed 'managed boxes' or 'managed pointers' - the pointer type preceded with the `@` sigil - have been deprecated and placed behind a feature gate. Although you may regain access to these types by applying the `#[feature(managed_boxes)];` attribute to a crate, managed boxes will be completely removed very soon, so all code should begin transitioning to the `Rc` or `Gc` types now included in the standard library.

`Rc` is a reference counted pointer located in `std::rc`, and `Gc` is *intended* to be a garbage collected pointer, located in `std::gc`. `Gc` is currently just a wrapper around `@`, which is reference counted.

Here's an example of using `Rc` and `Gc`.

```
use std::rc::Rc;
use std::gc::Gc;

fn main() {
    let rc1 = Rc::new(1);
    let rc2 = rc1.clone();
    println!("{}", *rc1.borrow() + *rc2.borrow());

    let gc1 = Gc::new(1);
    let gc2 = gc1.clone();
    println!("{}", *gc1.borrow() + *gc2.borrow());
}
```

Note that to get the value inside one of these boxes you first call the `borrow()` method, which returns `&T`, then dereference *that*. This is indeed verbose, but in the future Rust will provide an overloadable dereference operator to make these operations easier.

In addition to the deprecation of `@`, `@mut` has been completely removed from the language. The new `Cell` and `RefCell` types in `std::cell` can be used to introduce interior mutability to any types; `Gc<RefCell<T>>` is the equivalent of `@mut T`. The difference between `Cell` and `RefCell` is that `RefCell` features dynamic borrowing, by which the interior can be borrowed with the `borrow` method, and additional borrows are prevented dynamically at runtime. The `Cell` type in contrast only allows access through a copying getter and mutation through a setter; it has no dynamic borrowing, but is limited to POD types (plain old data). In practice, we've found that dynamic borrowing is a common source of errors when dealing with mutable types with complex lifecycles, so it's probably preferable to use `Cell` over `RefCell` when possible.

```
let x = Cell::new(10);
assert_eq!(x.get(), 10);
x.set(20);
assert_eq!(x.get(), 20);

let y = Cell::new((30, 40));
assert_eq!(y.get(), (30, 40));
```

```
let x = RefCell::new(0);
x.with_mut(|x| *x += 1);
let b = x.borrow();
assert_eq!(1, *b.get());
```

This change again makes dealing with non-owned boxes more verbose. The next few releases will involve a lot of continued work overhauling how pointers work in Rust, so expect to see improvements to usability in the near future.

### Changes to closures

Closures have changed extensively, on the surface. `~fn`, `&fn`, and `@fn` have all been removed. `&'a fn()` is now written `'a ||`. That is, closure types now look like their declaration. Another example, `&fn(int, int) -> uint` is now `|int, int| -> uint`. Additionally, plain functions (those without an environment) can now be written as `fn()` rather than `extern fn()`. `fn(int, int) -> uint` is now a valid type.

Another function type, `proc()` has been added. This represents functions which can only be called once, and are similar to `~once fn`. Their environment is stored on the heap. One can move into a `proc`. They are primarily used for task bodies. `do` now works exclusively with `proc`s. Example: `proc(int, int) -> uint`. A longer example:

```rust
use std::io::File;
use std::io::buffered::BufferedReader;

fn main() {
    let f = BufferedReader::new(File::open(&Path::new("rust_is_awesome.txt")).unwrap());
    do std::task::spawn() {
        let mut f = f; // this is a move!;
        let l = f.read_line();
        println!("{}", l);
    }
}
```

Uses of the other closure types (`~fn`, `@fn`) should instead use trait objects.

todo: future unboxedness

### Changes to libraries and linkage

This release brought many changes to how rust libraries and native libraries link together. The compiler now supports a much larger set of functionality when dealing with static and dynamic linkage with both rust and native libraries.

#### Naming a rust crate

The `#[link]` attribute at the *crate level* has been deprecated in favor of the `#[crate_id]` attribute. Crates should now name themselves like:

```rust
#[crate_id = "foo#0.9"];
```

This indicates that the crate is named `foo` with a version number of `0.9`. Some other examples of crate attributes are:

| `crate_id` | path | name | version |
|------------|------|------|---------|
| `foo`      | foo  | foo  | 0.0     |
| `foo#1.0`  | foo  | foo  | 1.0     |
| `github.com/foo/bar;foo#1.0`  | github.com/foo/bar | foo  | 1.0     |

#### Linking Rust crates

Before 0.9, generating a rust library meant that a dynamic library was always generated, and all rust crates were linked to one another dynamically. As of 0.9, the compiler can now generate static rust libraries so that there will be no dynamic rust dependencies. Detailed documentation can be found [in the rust manual](http://static.rust-lang.org/doc/master/rust.html#linkage).

There are now more flavors of rust crates which can be generated, summarized here:

* `bin` - an executable
* `dylib` - a dynamic rust library
* `rlib` - a static rust library. An `rlib` is Rust's version of a `libfoo.a` file, and is used to statically link libraries together. 
* `staticlib` - a static archive used to integrate rust into an existing application

Each of these flavors can be passed to the compiler via flags (`--bin`, `--rlib`, etc), and they may also be an attribute inside the crate (`#[crate_type = "staticlib"]`). The compiler can generate multiple outputs simultaneously if more than one format is specified.

#### Linking native libraries

Native libraries are now supported as first-class citizens in rust. Usage of the `link_args` attribute to link native libraries is deprecated and now behind a feature gate (`feature(link_args)`). Native libraries are now linked with a `#[link]` attribute (different than the deprecated crate version) on `extern` blocks. Examples are shown below:

```rust
// Dynamically link to a native library called "foo"
#[link(name = "foo")]
extern {
    fn foo_function();
}

// Statically link to a native library called "bar"
#[link(name = "bar", kind = "static")]
extern {
    fn bar_function();
}

// Dynamically link to an OSX framework called "baz"
#[link(name = "baz", kind = "framework")]
extern {
    fn baz_function();
}
```

With the advent of linking to static native libraries, it is now possible to have a native library that is purely a build dependency and need not be distributed alongside the rust library.

### Runtime improvements and I/O

The rust runtime has received yet another round of lots of love for the 0.9 release. You'll find that lots of things have been moved around, but all functionality should still exist in one form or another. Major components of the 0.9 release was a complete I/O overhaul and a complete `std::comm` (channels) overhaul.

#### I/O movement

All I/O now lives in the top-level `std::io` module with sub-functionality underneath it. This is mostly a movement of the old `std::rt::io` to `std::io`, but some interfaces have been improved and tweaked along the way.

Most of the "filesystem functionality" in 0.8 was spread out across std::os, std::rt::io, std::path and friends. This has now all been consolidated into std::io::fs. For green threads, none of this continues to use a blocking implementation (it's all an appropriate libuv implementation).

#### `std::comm` rewrite

In addition to an I/O update, the `std::comm` interface has been completely overhauled and redone. You'll find that all the old traits are gone, oneshots are gone, and `SharedChan` is no longer constructed from `Chan`. The new channels are much faster than their older counterparts, and the constructors are now done through the types rather than top-level functions.

```rust
let (p, c) = Chan::new();
c.send(());
p.recv();

let (p, c) = SharedChan::new();
let c2 = c.clone();
c2.send(());
p.recv();
```

You'll also find that the `std::comm::select` module supports selection over channels, but the interface is not fully baked yet. You can see an example usage of the macro defined in that module for using `select!`, but it is not currently available to all programs.

A major improvement of channels from before is that they now work seamlessly in and out green and native modes. You must have a local rust Task to receive on a channel (that's how it knows to block), but you can send on a channel from essentially any context.

#### libgreen vs libnative

All I/O implementations have been refactored into two separate libraries, libgreen and libnative. The green-threading library (libgreen) is a libuv-backed implementation of all I/O (just refactored outside of libstd). The native-threading library (libnative) is a pthread/libc-backed implementation of all I/O (much of it new code).

You can explicitly link to one or the other in order to use I/O handles, but this is not recommended. If a library or application is compiled against libstd, then it will work seamlessly among native and green threads with no modifications.

Details on how to use libgreen and libnative can be found in a [mailing list post](https://mail.mozilla.org/pipermail/rust-dev/2013-December/007565.html), and more extensive documentation will be available for the next release (a few portions are still in flux).

For now, libgreen is still used as the default for booting programs, but it is planned that for the 1.0 release of Rust to use libnative by default.

#### `start_on_main_thread`

Users of the old `start_on_main_thread` function should now use code similar to this:

```rust
extern mod native;

#[start]
fn start(argc: int, argv: **u8) -> int {
    do native::start(argc, argv) {
        main();
    }
}

fn main() {
    // This function is running on the main native thread
}
```

Keep in mind that not all I/O is implemented for native threads at this time. Missing components are timers, signals, unix sockets, and DNS.

Note that this code will probably look different in the future (opting into a main-thread libnative startup).

## 0.8 September 2013

### `for` loop and `Iterator`

The closure-based `for` loop syntax has been replaced with one based on iterator objects. Iterator objects were already widely in use by Rust 0.7, so for most code the only difference will be the lack of a call to the `advance` compatibility method.

Rust 0.7:

```rust
let mut xs = [1, 2, 3];
let ys = [4, 5, 6];
for xs.mut_iter().zip(ys.iter()).advance |(x, y)| {
    *x = *y + 5;
}
```

Rust 0.8:

```rust
let mut xs = [1, 2, 3];
let ys = [4, 5, 6];
for (x, y) in xs.mut_iter().zip(ys.iter()) {
    *x = *y + 5;
}
```

If there is still a function using the old iterator protocol, you can manually pass a closure or use a `do` loop, as a stopgap until it is rewritten as an iterator:

```rust
do old_iteration_function() |_x| {
    true
};
```

For more information, see the [iterator tutorial](http://static.rust-lang.org/doc/master/tutorial-container.html#iterators).

### New string formatting with `format!`

`format!` is a new macro for formatting strings, to eventually replace `fmt!`. The main reasons for this change were:

* `format!` is much higher performance than `fmt!`, namely 0 allocations are performed
* `format!` has less code bloat at callsites (and generally on object size)
* `format!` uses traits to format arguments, meaning that formatting is extensible to all types
* `format!` was designed with future i18n efforts in mind, leading to the newer syntax.
* `format!` supports formatting into generic `rt::io::Writer` streams instead of only to strings

Some examples of the new formatting syntax are:

```rust
format!("Hello")                  // => ~"Hello"
format!("Hello, {:s}!", "world")  // => ~"Hello, world!"
format!("The number is {:d}", 1)  // => ~"The number is 1"
format!("{:?}", ~[3, 4])          // => ~"~[3, 4]"
format!("{value}", value=4)       // => ~"4"
format!("{} {}", 1, 2)            // => ~"1 2"
```

This change also means that other macros based on `fmt!` will be migrating to `format!` syntax soon. None of the macros were removed from 0.8, but they will likely be removed in the next release. In the meantime, new macros were introduced using the `format!` syntax by appending a `2` at the end of the name (`debug2!()`, `fail2!()`, `info2!()`, etc.). The `assert!` macro does not have a `format!` equivalent, it still used the old `fmt!` syntax.

Additionally, a new macro, `format_args!` was added. This can be used to support `format!`-style logging/printing without requiring an allocation to be performed. The idea of this macro is to create a `va_args`-like struct which is opaque to the user but validated at compile-time. This is then passed around and can eventually be handed back to `std::fmt` to print/create a string. usage of this macro can be found in the documentation.

Extensive documentation about this syntax extension can be found in the [module's header](http://static.rust-lang.org/doc/master/std/fmt/index.html).

### FFI changes

The FFI was changed to be more efficient, introducing first-class foreign function pointers. All calls to foreign functions now happen directly, whereas previously they were called through a wrapper function that switched to a special C stack before the call. In order to ensure that there is enough stack space available to run foreign code, any function that calls a foreign function must be annotated with the `#[fixed_stack_segment]` attribute. This attribute ensures that the function executes with a large amount of stack available.

```
extern {
    fn a_foreign_fn();
}

#[fixed_stack_segment]
fn a_caller_of_a_foreign_fn() {
    a_foreign_fn();
}
```

Strategic placement of the `fixed_stack_segment` attribute can make FFI calls very efficient at the expense of extra annotations. When performance is not a concern, the `externfn!` macro can be used to declare the foreign function and the fixed-segment wrapper at once.

```
externfn!(fn a_foreign_fn());
```

Because they are now called directly, C foreign functions now have the type `extern "C" fn` and can be passed by value and called from Rust code:

```
let f: extern "C" fn() = a_foreign_fn;
f();
```

### Cast naming conventions

Functions for converting between types have been renamed (or added) to follow a new convention:
- `as`: cheap conversions that are normally just converting a reference to a different type, but not changing the in-memory representation, e.g. `string.as_bytes()` gives a `&[u8]` view into a `&str`. These don't consume the convertee.
- `to`: expensive conversions that may allocate and copy data, e.g. `string.to_owned()` copies a `&str` to a new `~str`. These also don't consume the convertee.
- `into`: conversions that consume the convertee and almost always don't allocate new memory (they may change the in-memory representation, but will normally do so in-place), these are cheaper than `to` conversions, e.g. `string.into_bytes()` converts a `~str` to a `~[u8]` without copying.

These are not hard rules, since generically implemented functions mean that an `into` conversion that doesn't copy/allocate is impossible for some types, e.g. `string.into_owned()` (from the `Str` trait) doesn't copy when `string` is `~str`, but it is forced to when `string` is `&str` or `@str`.

### New runtime and experimental I/O subsystem

The Rust runtime, which includes the task scheduler, previously written in C++, was rewritten in Rust, making Rust almost completely self-hosting. For the most part this does not have visible changes, though there are a few regressions in message passing functionality (particularly related to 'select' - receiving on multiple ports) and lurking bugs. The largest regression is the temporary removal of segmented stacks. For the time being all Rust tasks run with large stacks and *no overflow protection*. This means that it is possible to cause segfaults and other odd behavior by recursing deeply.

The runtime rewrite was performed in order to facilitate a new I/O subsystem that integrates an I/O event loop such that I/O does not block the task scheduler. This I/O system is temporarily located in `std::rt::io` and implements TCP, UDP, files, timers, and process spawning. It is incompatible with the old, still-used `std::io`, but will replace it in the future, and should be considered experimental.

Related links:
  * Turning on the runtime: https://mail.mozilla.org/pipermail/rust-dev/2013-August/005158.html
  * Scheduler rewrite issue: https://github.com/mozilla/rust/issues/4419

### This week in Rust

For even more information about what happened during this release cycle, see the past editions of 'This Week in Rust'.

* http://cmr.github.io/blog/2013/07/13/this-week-in-rust/
* http://cmr.github.io/blog/2013/07/21/this-week-in-rust/
* http://cmr.github.io/blog/2013/07/29/last-week-in-rust/
* http://cmr.github.io/blog/2013/08/04/this-week-in-rust/
* http://cmr.github.io/blog/2013/08/10/this-week-in-rust/
* http://cmr.github.io/blog/2013/08/19/this-week-in-rust/
* http://cmr.github.io/blog/2013/08/25/this-week-in-rust/
* http://cmr.github.io/blog/2013/08/31/this-week-in-rust/
* http://cmr.github.io/blog/2013/09/07/this-week-in-rust/
* http://cmr.github.io/blog/2013/09/15/this-week-in-rust/
* http://cmr.github.io/blog/2013/09/23/this-week-in-rust/

## 0.7 July 2013

This release didn't have many breaking language changes that require lengthy explanation beyond the short release notes, but there were many improvements to the libraries this time, and it feels like the standard library is beginning to reflect the final design.

### Cloning vs. copying

The `copy` keyword is being removed in favor of traits. Explicit copying is now performed with the `clone` method of the `Clone` trait, which can automatically be derived with `#[deriving(Clone)]`. `Clone` is part of the Rust prelude so is always in scope.

```rust
#[deriving(Clone)]
struct PopsicleToken;

let my_token = PopsicleToken;
let your_token = my_token.clone();
```

`Clone` makes shallow copies of managed pointers and other shared pointer types like `Rc` and `ARC`. For deep copies use the `DeepClone` trait and the `deep_clone` method.

Note that `copy` has not been removed from the language yet and there are still some types like fixed-size arrays and tuples without `Clone` implementations.

### Iterators

Rust is in the middle of a transition to a new iterator mechanism. Instead of using higher-order functions like Ruby ("internal iterators") Rust will use `Iterator` types, like Java ("external iterators"). There was an [excellent blog post][iterators] recently exploring the pros and cons of each. External iterators are more flexible and their code is believed to be faster to generate.

Note that the `for` protocol is still using the old iteration protocol, but it will be updated in the next release. For compatibility with `for`, `Iterator`s have an `advance` method that converts them to the old-style.

```rust
let v = [0, 1];
for v.iter().advance |i| { ... }
```

There is a new [tutorial][containers] on this topic.

Further reading:
* https://mail.mozilla.org/pipermail/rust-dev/2013-June/004364.html
* https://mail.mozilla.org/pipermail/rust-dev/2013-June/004599.html

[iterators]: http://journal.stuffwithstuff.com/2013/01/13/iteration-inside-and-out/
[containers]: http://doc.rust-lang.org/doc/0.7/tutorial-container.html

### Numeric traits

Rust now has a proper numerical tower, defined in [`std::num`](http://static.rust-lang.org/doc/0.7/std/num.html). All numeric types implement trait `Num`, which in turn inherits from the operator traits, `Neg`, `Add`, `Eq`, as well as the `Zero` and `One` traits. `Num` itself defines no methods.

```rust
pub trait Num: Eq + Zero + One
             + Neg<Self>
             + Add<Self,Self>
             + Sub<Self,Self>
             + Mul<Self,Self>
             + Div<Self,Self>
             + Rem<Self,Self> {}
```

`Signed` types extend `Num` with methods specific to signed types, such as `abs`, and there's a corresponding `Unsigned` trait (that defines no new methods).

The `Real` trait then defines the the interface required of real numbers, which inherits from `Signed` as well as a number of traits that define common math operations: `Fractional`, `Algebraic`, `Trigonometric`, and `Hyperbolic`. `RealExt` further defines less common operations, the gamma and bessel functions.

Then there are a number of traits specific to primitive numeric types:

* `Bounded` types have a minimum and maximum value.
* `Bitwise` inherits from the bit-twiddling operators, `BitAnd`, `BitOr`, `BitXor`, `Shl`, `Shr`, as well as `Not`, and further defines several methods for counting and querying bits.
* `Primitive` again collects a number of traits that are implemented by the primitive types.
* `Int` and `Float` traits define the interface to machine integers and floats.

Furthermore, `Add`, `Sub`, `Mul`, and `Neg` no longer include [compound assignment](https://en.wikipedia.org/wiki/Compound-assignment) in favor of the upcoming `AddAssign`, `SubAssign`, etc traits. For now, ported code shall take the form of `foo = foo + bar` instead of `foo += bar`.

### rustpkg

While rustpkg is still in an experimental state, there are a number of improvements; see the rustpkg manual for more details.
* rustpkg uses a URL-like package ID to specify a local or remote package, and has the ability to download remote packages from github.
* rustpkg infers package IDs from directory structure; packages need no longer declare their identity explicitly
* Package IDs can have explicit versions attached.
* Package scripts (pkg.rs) are no longer required; rustpkg infers crates to build based on reasonable defaults.
* Package scripts can explicitly invoke the default build logic.
* rustpkg requires a specific directory structure for workspaces.
* rustpkg infers dependencies from `extern mod` directives, and doesn't require `-L` flags on the command line for finding libraries.

## 0.6 April 2013

This was a very busy development cycle focused on completing as many of the planned language-level changes as possible in the release time-window. While we cannot promise that this is the last time there will be incompatible changes, the great majority of anticipated language-level changes are complete in this version. We expect subsequent releases before a beta and final 1.0 to be more focused on non-language-level work (performance, libraries, packaging and building, runtime system) with only modest language-level changes as we discover bugs and areas requiring residual polish (primarily in the trait system, macro system, and borrow check).

### Minor syntax changes

  - The `self` type parameter in traits was renamed `Self` (capital) to help differentiate it from the keyword `self`
  - The _implicit_ `self` parameter in trait methods is now deprecated. Methods should now specify their `self` parameter explicitly.
  - The `Durable` trait was removed; it is synonymous, as a type-bound, with the `static` lifetime.
  - Trailing sigils on closure types such as `fn@`, `fn~` and `fn&` were removed in favour of the more-consistent leading sigils `@fn`, `~fn` and `&fn`
  - The `const` keyword was renamed to `static`, to accommodate for future unsafe mutable static variables.
  - The `export` keyword, formerly deprecated, was removed; `pub use exported::path;` is synonymous.
  - The `move` keyword was removed; owned types are always passed and assigned by moving now.
  - The `fail` and `assert` keywords were replaced with macros `fail!()` and `assert!()`.
  - The `log` keyword was removed; use `debug!()`, `error!()` and similar macros.
  - Single-element tuples, denoted `(T,)` were added as a special-case to help with some macros.
  - "Newtype" enums (those with a single variant) such as `enum Foo = int;` were removed; use tuple structs such as `struct Foo(int);` instead
  - The visibility modifiers (`pub` and `priv`) are no longer permitted on trait implementations. They were redundant with the visibility modifiers on trait and type declarations themselves.
  - The `#[deriving_eq]` attribute (and the related ones for `clone` and `iter_bytes`) was removed; use `#[deriving(Eq)]` instead. Multiple traits can be specified in the same attribute, as in `#[deriving(Eq, Clone, IterBytes)]`.

#### Inherited mutability
The `mut` keyword is no longer permitted in `~mut T`, `[mut T]`, or in fields of structures; in all cases mutability is controlled by mutability of the owner (inherited mutability). So things written like this in 0.5:
```
struct Foo {
   mut x: int,
   mut y: int
}
fn main() {
  let z = Foo { x: 10, y: 11 };
  z.y = 12;
}
```
must now be written like this in 0.6:
```
struct Foo {
   x: int,
   y: int
}
fn main() {
  let mut z = Foo { x: 10, y: 11 };
  z.y = 12;
}
```
This change permits reasoning about mutability of an owned structure by reasoning about the mutability of its owner, which in turn means many structures support "freezing" and "thawing": acquiring mutator-methods (statically) when held in a mutable owner, and losing those methods (statically) when moved to an immutable owner.

#### trait bounds are separated by `+`

A function like this in 0.5:

```
fn foo<T:X Y Z, U:Q>() {
}
```
is written like this in 0.6:

```
fn foo<T:X + Y + Z, U:Q>() {
}
```

#### Trait impls use `for`

A trait implementation that was written this way in 0.5:
```
impl Ty : Trait {
}
```
is written this way in 0.6:
```
impl Trait for Ty {
}
```

### Foreign module changes

Foreign modules (those declared `extern`) have been simplified to anonymous `extern` blocks, and all `extern` functions are now unsafe. This means that code like this in 0.5:
```
extern mod libc {
    fn getc() -> c_char;
}
fn main() {
    let x = libc::getc();
}
```
must be written this way in 0.6:
```
extern {
    fn getc() -> c_char;
}
fn main() {
    unsafe {
        let x = getc();
    }
}
```

### Vector pattern-matching enhancements

Vectors can now be matched against any combination of prefix, suffix and variable-length remainder patterns. The vector is destructured into the variable names given: the variable-length remainder is bound to a slice:

  - `[a, b..]` to match a vector prefix
  - `[..a, b]` to match a vector suffix
  - `[a, ..b, c]` to match both prefix and suffix

### Structural records removed

This release completes the consolidation of older "structural records" (those lacking a constructor name, denoted directly as `{field: type, ...}`) with the newer "nominal structs". Old code written like this in 0.5:
```
type Cat = {
    name: ~str,
    color: Color
};

fn main() {
    let x = { name: ~"fluffy", color: Grey };
}
```
Must be written this way in 0.6:
```
struct Cat {
    name: ~str,
    color: Color
}

fn main() {
    let x = Cat { name: ~"fluffy", color: Grey };
}
```


### Changes to the borrow check and borrowed pointers

  - Borrowing a mutable managed pointer (i.e. passing `@mut T` to an argument of type `&T` or `&mut T`) changed. The borrow is now performed with a dynamic (runtime) write-barrier check that can fail if multiple borrows are performed on the same value. In other words, the type `@mut T` now behaves like the library module `Mut<T>` did in 0.5. This change means that a class of potential errors -- multiple mutable aliases of the same memory -- that was formerly prohibited by a conservative static rule in 0.5 will be left to a dynamic check at runtime in 0.6. Experience with the rule present in 0.5 and before demonstrated that the static rule was too restrictive to be useful in practice. We will monitor the resulting semantics carefully to see if the new dynamic failure mode occurs frequently in practice.

### Further adjustments to name resolution

  - `use` statements are now crate-relative by default, meaning that they resolve from the "top" of the crate.
  - As a specific corollary: "chained" `use` statements -- those that refer to abbreviated names when finding targets -- are no longer legal. For example: `use std::libc; use libc::raw;` must now be written `use std::libc; use std::libc::raw;`.
  - The `super` and `self` can be used in paths to refer to parent modules and the current module, respectively.
  - `extern mod` statements must occur before `use` statements, and `use` statements can now shadow `extern mod` statements (rather than vice-versa, as was the case in 0.5). This change makes the shadowing behavior between `extern mod`, `use` and module-local item definitions consistent with the (required) order of writing them.

### Minor semantic changes

  - the nil type `()` is now properly 0 bytes
  - the "main function" of an executable crate -- where execution begins -- does not need to be called `main` anymore. While `main` is the default, any function marked with the attribute `#[main]` will override this behavior.
  - The default type of an inferred closure (such as `|x| x+1`) is now `&fn` rather than `@fn` as it was in 0.5.

### Inline Assembly

Rust now supports using inline assembly through the `asm!` macro. The syntax roughly matches that of gcc/clang:

```
asm!(assembly template
   : output operands
   : input operands
   : clobbers
   : options
   );
```

`asm!` must be wrapped in an `unsafe` block.

**Assembly Template**

The `assembly template` is the only required parameter and must be a literal string (i.e `""`)

```
asm!("NOP");
```

Output operands, input operands, clobbers and options are all optional but you must add the right number of `:` if you skip them:

```
asm!("xor %eax, %eax"
    :
    :
    : "eax"
   );
```
Whitespace also doesn't matter:
```
asm!("xor %eax, %eax" ::: "eax");
```

**Operands**

Input and output operands follow the same format: `: "constraints1"(expr1), "constraints2"(expr2), ..."`. Output operand expressions must be mutable lvalues:

```
fn add(a: int, b: int) -> int {
    let mut c = 0;
    unsafe {
        asm!("add $2, $0"
             : "=r"(c)
             : "0"(a), "r"(b)
             );
    }
    c
}
```

**Clobbers**

Some instructions modify registers which might otherwise have held different values so we use the clobbers list to indicate to the compiler not to assume any values loaded into those registers will stay valid.
```
// Put the value 0x200 in eax
asm!("mov $$0x200, %eax" : /* no outputs */ : /* no inputs */ : "eax");
```

Input and output registers need not be listed since that information is already communicated by the given constraints. Otherwise, any other registers used either implicitly or explicitly should be listed.

If the assembly changes the condition code register `cc` should be specified as one of the clobbers. Similarly, if the assembly modifies memory, `memory` should also be specified.

**Options**

The last section, `options` is specific to Rust. The format is comma separated literal strings (i.e `:"foo", "bar", "baz"`). It's used to specify some extra info about the inline assembly:

Current valid options are:

1. **volatile** - specifying this is analogous to `__asm__ __volatile__ (...)` in gcc/clang.

2. **alignstack** - certain instructions expect the stack to be aligned a certain way (i.e SSE) and specifying this indicates to the compiler to insert its usual stack alignment code

3. **intel** - use intel syntax instead of the default AT&T.

## 0.5 December 2012

This was a fairly slow development cycle that focused on implementing more trait features, such as trait inheritance and static methods, with the goal of enabling more expressive standard libraries. This version more-or-less completes Rust's long transition to a linear type system, with non-copyable types moving automatically (the `move` keyword is deprecated).

### Explicit self

Work continues on requiring that instance methods always use explicit self-type declarations, and now `self`, `&self`, `~self`, and `@self` are all valid self types and should work. Methods with explicit self are declared like `fn foo(self, arg1: Type1, arg2: Type2) { ... }`. The old method declaration syntax, without a self type, is deprecated and will likely be removed in 0.6.

```
struct MyType { ... }

impl MyType {
    fn i_need_a_managed_box(@self) { ... }
    fn i_need_an_owned_box(~self) { ... }
}

let managed_value = @MyType { ... };
managed_value.i_need_a_managed_box();
```

Self types also have correct support for move semantics, enabling some nice patterns for unique types.

```
enum Option<T> { Some(T), None }
impl<T> Option<T> {
    fn unwrap(self) -> T {
        // Extract the value from the option and return it.
        // There is no copying here as these operations all
        // move by default now for unique types.
        match self {
            Some(v) => v,
            None => fail
        }
    }
}

fn maybe_get_value() -> Option<MyType> { ... }

let value = maybe_get_value().unwrap();
```

### Trait inheritance

Traits may now declare supertraits. When a trait has supertraits then any type which implements the subtrait must also implement the supertrait. Trait inheritance allows a group of traits to be treated as a single trait in bounded type parameters.

```
trait Add {
  fn add(&self, other: &self) -> self;
}
trait Sub {
  fn sub(&self, other: &self) -> self;
}
trait Num: Add Sub { }
```

The methods of each trait must still be implemented seperately.

```
impl MyNumType: Add {
  fn add(&self, other: &MyNumType) -> MyNumType { ... }
}
impl MyNumType: Sub {
  fn sub(&self, other: &MyNumType) -> MyNumType { ... }
}
impl MyNumType: Num { }
```

Then the constrained trait may be used in type parameter constraints in place of its supertraits.

```
fn calculate<T: Num>(val1: T, val2: T, val3: T) -> T {
  val1.add(&val2).sub(&val3)
}
```

Note that supertrait methods are not yet available on subtrait 'object' types.

```
// This doesn't work yet
let t = &MyNumType { ... };
let num = t = &Num;
num.add(othernum);
```

Furthermore, trait inheritance is not yet aware of the "kind" traits, so using `Copy`, etc. as supertraits will not work as expected.

### Static method resolution

In 0.4 static methods 'leaked' out of the traits in which they were defined and were resolved in the outer module scope. This would mean that two sibling traits couldn't define a static method with the same name.

```
// Couldn't do this before because `bar` actually exists in the outer scope
trait Foo<T> { static fn bar() -> T; }
trait Baz<T> { static fn bar() -> T; }

fn quux<T>() -> T {
    return bar();
}
```

In 0.5 the module and type namespaces are merged and static methods are resolved under the name of the trait in which they are defined.

```
// Now this is valid
trait Foo<T> { static fn bar() -> T; }
trait Baz<T> { static fn bar() -> T; }

fn quux<T>() -> T {
    // Now trait names may be used in paths to access static methods
    return Foo::bar();
}

// Likewise, static methods on types (anonymous traits) may be accessed via the type name
struct Swizzle { ... }
impl Swizzle { static fn bar() -> T; }

fn swozzle<T>() -> T {
    return Swizzle::bar();
}
```

### Path resolution

Path resolution has changed significantly to keep in-scope identifiers from leaking from parent modules to submodules. The old behavior, particularly when dealing with large projects with many files, made knowing what was in scope very difficult and prone to error.

In response, we're making a change to how imports (`use` statements) are resolved. They are now resolved relative to the top of the crate by default, and can be thought of as absolute paths through the crate's module namespace. Paths may be prefixed with the contextual keywords `self` and `super` to modify how the lookup is performed.

```
extern mod std;

mod foo {
    // Bring `net` into scope
    use std::net;

    fn f() { ... }

    // This submodule no longer inherits the scope of the outer module
    mod bar {
        // Need to import `net` again to use it
        use std::net;
    }

    mod baz {
        // We can also use `super` to get at `foo`s import
        use super::net;
    }

    mod quux {
        // Can also use `self` to import from child modules
        use self::boo::net;

        mod boo {
            pub use std::net;
        }
    }
}
```

*Note: There will be related changes to path resolution in other contexts as well in the next release*

*Note: `self` and `super` will likely be promoted to full keywords (not contextual keywords) in the next release*

### Automatically-derived trait implementations

Implementations of the `Eq` and `IterBytes` can be automatically derived using syntax extensions (types that implement `IterBytes` can automatically implement `Hash`, so can be used in hash tables).

```
#[deriving_eq]
#[deriving_iter_bytes]
struct Foo {
  bar: int
}
```

This should work on all struct and enum types, and does what you would likely expect, delegating to the corresponding impls of each subcomponent in turn.


### Condition handling

This release adds a new API for dealing with errors, `core::condition`. Unlike the concept of *exceptions* in other languages, *conditions* are handled at the site where they are raised rather than the site where they are caught. Failure to handle a condition results in task failure.

```
// A condition has a name, input type and output type.
// The input type contains details of the condition, passed to a handler.
// The output type is what the handler will return if it handles the condition.
condition! {
    missing_input_file : Path -> Reader;
}

// Install the handler
do missing_input_file::cond.trap(|pathname| {
    // Handle a missing input file by providing a fake empty reader.
    io::BytesReader { bytes: bytes, pos: 0u } as Reader
}).in {
    // The condition handler is valid over the dynamic extent
    // of the block passed to trap.
    foo(&Path("/nonexistent"))
}

fn foo(p: &Path) {
    let r = if p.exists() {
        io::file_reader(p)
    } else {
        // If there's a handler in one of our callers, this
        // will evaluate to the handler's return value and
        // we will carry on, no unwinding.
        missing_input_file::cond.raise(p)
    }
    let v = r.read();
    // ...
}

```

The standard library has not yet been updated to make use of conditions.

### Other important changes

The `Send` trait, one of the built-in 'kinds', is now called `Owned`. `Owned` types contain no managed or borrowed pointers. The little-known trait previously called `Owned` is now called `Durable`. `Durable` types contain no borrowed pointers (though in the future they will probably allow borrowed pointers to the `static` region). All `Owned` types are `Durable`.

The declarative language for .rc files has been [removed](https://mail.mozilla.org/pipermail/rust-dev/2012-December/002679.html). The biggest practical implication of this change is that, for projects using 'companion modules',
a `project.rc` combined with `project.rs`, the companion module (`project.rs`) will not be loaded automatically. You should copy the contents of `project.rs` into `project.rc` and delete `project.rs`.

The `move` keyword is no longer needed under normal circumstances and should be considered deprecated. Types that are not implicitly copyable now move by default.

## 0.4 October 2012

This version was focused on completing as many disruptive syntax changes as possible, including changing and reducing keywords, beefing up traits and removing classes, changing how imports and exports are handled, and moving to a camel case convention for types.

Borrowed pointers matured and replaced argument modes in some of the libraries. A number of new concurrency features were added.

### Camel cased types

We have a new convention that requires that types (and enum variants, which are not yet types) be camel cased. The entire core and standard libraries have been converted. We generally prefer that acronyms not be written with all caps, so e.g. the standard URL type is written `Url`.

There is a lint check for this called `non_camel_case_types` but it is disabled by default.

### Keywords

This release tried to narrow down the set of keywords and the current keywords are expected to be close to final.

Keyword changes:

* `iface` became `trait`
* `ret` became `return`
* `alt` became `match`
* `again` was removed in favor of reusing `loop` to continue, i.e. `loop { if foo { loop; } }`
* `import` and `export` were removed in favor of `pub` and `priv` item-level visibility (see below).
* `class` was removed in favor of combinations of `struct`, `impl` and `trait`

The current keywords are as follows.

```none
as assert
break
const copy
do drop
else enum extern
fail false fn for
if impl
let log loop
match mod move mut
priv pub pure
ref return
self static struct
true trait type
unsafe use
while
```

Notes:
* `be` is not a keyword, but is reserved for [possible future use](https://github.com/mozilla/rust/issues/217).
* `self` and `static` are currently parsed as contextual keywords, but are expected to not be keywords in the future.
* [`assert`](https://github.com/mozilla/rust/issues/2228), [`log`](https://github.com/mozilla/rust/issues/554), and [`fail`](https://github.com/mozilla/rust/issues/2232) are likely to be converted to macros.
* `drop` may [become a trait](https://github.com/mozilla/rust/issues/3061) rather than a keyword, as might `const`.

### Structs replace classes

Classes were overhauled; the `class` syntax was removed from the language, in favor of method-less `structs` combined with method-bearing `impls`. The new `struct` syntax is very simple:

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

### Explicit self

We are in the process of generalizing traits so that they can include
both methods (which take a receiver) and functions (which do not).  To
that end, we have added a syntax called *explicit self*.  In the
explicit self system, methods are identified because their first parameter
is called `self` (similar to Python).  Optionally, the parameter can include a sigil
indicating what kind of pointer should be used:

```
struct Foo { ... }
impl Foo {
    fn method0(self, ...);       // self has type `Foo`
    fn method1(&self, ...);      // self has type `&Foo`
    fn method2(&mut self, ...);  // self has type `&mut Foo`
    fn method3(@self, ...);      // self has type `@Foo`
    fn method4(~self, ...);      // self has type `~Foo`
}
```

Explicit self can also appear in trait declarations.  Explicit self is
currently optional but is expected to become mandatory in a future
release.

### Static methods

Traits can now include methods that do not make use of a receiver.
Such functions are (currently) labeled with the keyword `static`.
Static functions are (currently) exported as functions in the module
which contains the trait where they were declared.  Here is an
example:

```
trait FromInt {
    static fn from_int(i: int) -> self;
}

struct Foo { v: int }
impl Foo: FromInt {
    static fn from_int(i: int) -> Foo {
        Foo { v: i }
    }
}

fn main() {
    let x: Foo = from_int(3);
    io::println(fmt!("%?", x));
}
```

In the future, when explicit self is mandatory, static functions will
be designated by the absence of a self receiver.  They are also
expected to become members of the trait itself, so you would write
`FromInt::from_int()`.


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

[macros]: http://static.rust-lang.org/doc/0.4/tutorial-macros.html

### Smaller syntax changes

* Region names are specified as `&r/T` instead of `&r.T`

### Trait-based operator overloading

Operator overloading is now expressed through a set of traits in the core library. The traits, as well as implementations on standard types can be found in `core::ops` and `core::cmp`. The set of overloaded operators has not changed, merely the mechanism used to define overloads.

### Hash function improvements

All hash functions in the standard library were replaced. Hashing is now statically dispatched (via traits) to an implementation of [SipHash](https://www.131002.net/siphash/), a fast keyed hash function. All hashtables are also now randomly keyed from Rust's CPRNG ([ISAAC](http://burtleburtle.net/bob/rand/isaacafa.html), seeded from the operating system PRNG on startup). In practice this should help defend Rust programs against complexity attacks.

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

let mut s1 = S1 { field: S2 { intfield: 1, boxfield: @2 } };

// s1 is in a mutable slot so we can mutate the fields
s1.field = S2 { intfield: 3, boxfield: @4 };

// We can also reach through owned boxes and mutate their insides
s1.field.intfield = 5;

// Managed boxes are not owned so their interior is not mutable
*s1.field.boxfield = 6; // ERROR
```

At first glance this might seem like an odd way to deal with mutability, but it seems to work very well with Rust's ownership semantics, and it enables some fascinating patterns that are particularly useful for concurrency. In particular, when combined with explicit self types it allows for 'dual mode' data structures that, under certain circumstances my mutate fields, and under others not, similar to C++ const methods.

```
struct S {
  foo: int
}

impl S {
  // Method declared with explicit mutable self type
  fn mutate(&mut self) {
    self.foo = 10;
  }

  fn do_not_mutate(&self) {
    // Can't mutate fields because we don't have a mutable self pointer
  }
}
```

So you can do neat stuff with this.

```
// LinearMap is an owned map type in core::send_map
// While it lives in a mutable slot we can call methods that mutate the map
let mut map = LinearMap();
map.insert(foo, bar);

// Move it into an immutable slot and the whole structure becomes 'frozen'
let map = move map;
map.insert(foo, bar); // ERROR

// At this point you can do some interesting things like put your entire map
// into a read-only shared-memory container.
let arc_map = ARC(move map); // ARC: 'atomically-reference-counted'

for repeat(50) {
  // Create another handle to the same map
  let arc_clone = arc::clone(arc_map);
  do spawn |move arc_clone| {
    // Read my immutable, shared hash map in parallel with other tasks
  }
}
```

### Task management

The task creation and lifecycle interface was significantly enhanced, adding more flexibility for grouping tasks according to which should die when a single task fails. There are now methods to spawn tasks with bidirectionally-linked, unidirectionally-linked, and unlinked failure propagation. There is also a low-level mechanism for storing task-local data, which will become the basis for some more advanced error handling mechanisms in future versions.

### Pipes

A [new communication system](https://github.com/eholk/rust/wiki/Proposal-for-channel-contracts) was added in this cycle, called "pipes". This is a lower-level, higher-performance system based on synchronous, 1:1 communication primitives with owned endpoints. Pipes are much friendlier to the memory subsystem of modern hardware and we expect to transition Rust's libraries to use pipes rather than the port/channel system that shipped in previous releases. This transition is not yet complete, but will happen in over subsequent releases.

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