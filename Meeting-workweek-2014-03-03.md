*Note: These are the notes and minutes for the work week from March 3-7, 2014 in Santa Cruz.
They are compiled from etherpads and are very incomplete.*

# Attending

* Alex Crichton
* Felix Klock
* Patrick Walton
* Brian Anderson
* Niko Matsakis
* Azita Rashed
* Dave Herman
* Nif Ward
* Nick Cameron
* Lars Bergstrom
* Huon Wilson (remote)
* Corey Richardson (remote)

# Agenda

The  primary objective of this work week is to make final decisions on all  remaining design questions impacting the release of Rust, address the  issues of highest risk.

The output of this week will be RFCs for major features, FAQs for difficult decisions (https://github.com/mozilla/rust/issues/4047), and possibly some rough documentation about The Rust Way.

We will take a lot of minutes for posterity.

## Topics

Each  topic has an advocate (in parens) who should be prepared to lead a  discussion on the topic, associated issue numbers, and expected output.

### High risk

* Alloc trait (nmatsakis / pcwalton) (RFC) (12038)
* Overloadable deref / index (nmatsakis / pcwalton) (RFC) (6515, 11780)
* GC (pnkfelix) (11778)
  - requirements of different GC and impact on language
* DST - how to implement? need work items (nrc / nmatsakis) (RFC) (work items) (6308)
* Strings and DST (pcwalton) (RFC) (6308?)
* Opt-in builtin traits (Send, Freeze, etc) (nmatsakis) (RFC)
* versioning (brson) (RFC)
  - what use cases do we need to support?
  - how are crates identified and distinguished?
  - under what scenarios can multiple versions coexist?
  - what goes into the crate hash, symbol hashes?
  - what happens during upgrades?
  - language-level versioning
  - what happens when crates require multiple versions of same crate?

### Mid risk

* pointer-sized integers (brson) (11831)
  - what does uint mean?
  - is there an inference default?
  - what types for indexing / shifting?
* linkage rules (acrichto) (docs)
* Attribute syntax (brson) (RFC) (2569)
* Operator overloading (nmatsakis) (RFC)
* self-type parameter syntax (nrc) (8888)
* Error handling policy (acrichto) (docs)
  * Fail unwind semantics vs. abort
  * What happens in a kernel? (__morestack alternatives, etc.)
  * `try!` and conversions between error types
* Uniform function call syntax (?) (11938)
  - Is this required for 1.0?
  - I mostly just want to make sure we have a way forward here

### Low risk

* Name resolution decidability (pcwalton)
  - define the algorithm
* Method self types and method resolution (nmatsakis, tied together)
* Trait resolution (nmatsakis)
* well-behaved iterators (brson)
  - https://mail.mozilla.org/pipermail/rust-dev/2014-February/008635.html
* requirements on unsafe code (brson / nmatsakis) (docs, The Rust Way)
* the rules for semicolons on tail expressions (nmatsakis)
  - explain rules, check that parser matches our belief
  - decide whether to keep rule on unit type, or maybe convert to lint
* ~ sugar 
* public struct fields & priv (acrichto)
* *mut (brson)

### Who knows

* packaging
  - too broad of a discussion for workweek?
* library API stabilization (brson)
  - what is the set of libraries we plan to stabilize?
* prelude (brson)
  - what's in it?
  - how to we do it without globs? can globs be simplified and made to work?
* logging - it's lame (brson)
* totalord/totaleq

# Decisions

*Note: this is the summary of decisions on various topics.
Detailed minutes of discussions follow later.*

## static/const

- Disallow taking the address of a non-mut static value unless it is Freeze
- Significant address policy is:
  - if static mut, yes
  - if static, no, unless tagged with some attribute or other
- Permit non-freeze types in statics
- References can be taken to `static mut` if the type is Share.

## allocators

* Implement this (https://etherpad.mozilla.org/Z9lc1uzBIz):
  - but don't use once fns yet

```
trait Alloc<T> : Clone {
    // Failure semantics?
    fn malloc(&self) -> *T;
    fn malloc_many(&self, n: uint) -> *T;
    
    // We will want to noodle around with the precise set
    // of free operations due to DST.
    fn free(&self, ptr: *T);
}

trait Box<T,P> { // "Smaht"
    fn box(&self, value: once || -> T) -> P;
}

// "System heap": libc malloc and free
struct DefaultAlloc;
impl<T> Alloc<T> for DefaultAlloc {
    fn malloc(&self) -> *T { ... }
    fn free(&self, ptr: *T) { ... }
}

// fixed type allocator
struct FixedTypeAllocator<T> { .. }
impl<T> Alloc<T> for FixedTypeAllocator<T> {
}

// arena allocator
struct MyArena { ... }
struct Arena<'a> { theArena: &'a MyArena }
impl<'arena,T> Alloc<T> for Arena<'arena> {
    ...
}

// Owned pointers
// box(OWNED) 22
// box(OwnedIn(myAllocator)) 22
struct Owned<T,A=DefaultAlloc> { box: *T, alloc: A }
struct OwnedIn<A>(A);
static OWNED = OwnedIn(DefaultAlloc);
impl<T,A: Alloc<T>> Box<T,Owned<T,A>> for OwnedIn<A> {
    fn box(&self, value: once || -> T) -> Owned<T,A> {
        let ptr: *T = self.alloc.malloc();
        // Cleanup: ptr
        ?? *ptr = value();
        Heap { box: ptr, alloc: self.alloc.clone() }
    }
}
impl<T,A: Alloc<T>> Drop for Owned<T,A> {
    fn drop(&mut self) {
        self.alloc.free(self.box);
    }
}

// Rc pointers
// box(RC) 22
// box(RcIn(myAlloc)) 22
struct Rc<T,A=DefaultAlloc>> { box: *RcBox<T,A> }
struct RcBox<T,A> {
    alloc: A,
    ref_count: Cell<uint>,
    value: T
}
struct RcIn<A>(A);
static RC: RcIn<DefaultAlloc> = RcIn(DefaultAlloc);
impl<T,A: Alloc<RcBox<T, A>>> Box<T,Rc<T,A>> for RcIn<A> {
    fn box(&self, value: once || -> T) -> Rc<T,A> {
        let ptr: *RcBox<T,A> = self.alloc.malloc();
        // Missing: cleanup ptr
        ptr.alloc = self.alloc.clone();
        ptr.refcount = Cell(1);
        ptr.value = value();
        Rc { box: ptr }
    }
}
```

## built-in traits

* rename Pod to Copy
* add Share
* remove Freeze
* Add Unsafe<T> (initially proposed as Mut<T>)
* Definition of built-in traits (see link)
* Don't do anything with #[deriving(Data)] yet
* opt-in

## struct fields/priv

* switch fields to priv by default
* remove private enum variants
* convert 'priv' to a reserved word
* tuple struct fields are private by default
* tuple struct fields take the pub keyword
* enum variant struct fields have to stay public
* identify structs that want to be priv tuple-structs and change them

## integer sizes

* array indexing needs to be strictly uints
* leave shifting taking any integer types

## logging 


* two variables affect logging, the global log level, and the local path
* global log level is a static mut in std, initialized to 255
* log!(...) emits code like the following

```
macro_rules! log(
    ($lvl:expr, $($arg:tt)+) => ({
        let lvl = $lvl;
        if lvl <= ::logging::GLOBAL_LOG_LEVEL {
            if ::logging::logging_enabled(my_module_string!()) {
                format_args!(|args| {
                    ::logging::log(lvl, args)
                }, $($arg)+)
            }
        }
    })
)


// logging.rs
pub static mut GLOBAL_LOG_LEVEL: uint = MAX_LOG_LEVEL;
static mut GLOBAL_LOG_MAP: *mut HashMap = 0 as *mut HashMap;
fn logging_enabled(lvl: uint, s: &str) -> bool {
    static INIT: Once = ONCE_INIT;
    INIT.doit(|| {
        GLOBAL_LOG_LEVEL = parse_log_level(getenv("RUST_LOG"));
        GLOBAL_LOG_MAP = parse_log_map(getenv("RUST_LOG_PATHS"));
        rt::at_exit(proc() { free(GLOBAL_LOG_MAP) });
    })
    
    if lvl > GLOBAL_LOG_LEVEL { return }
    global_map_contains(s)
}

local_data_key!(local_logger: ~Logger);
pub trait Logger {
    fn log(&mut self, lvl: uint, s: &str, arg: &fmt::Arguments);
}
```

* logging is initialized *lazily* the first time the log level is checked
* change logging spec to be more like
* remove __log_level and all logging machinery


## intrinsics

* get rid of "rust-intrinsic" in favor of "Rust"
  - tag intrinsics with "#[intrinsic]" or make a lang item
* transmute gets type checked specially, either on call or when taking value
  - both types must be known to have the same size
  - either is a non-pointer to generic then no
  - both must be sized or the same type
  - niko would not object to a warning about alignment
* reexport transmute, init, uninit, move_val_init, offset intrinsics instead of using wrappers
* cleanup rustdoc to indicate intrinsics nicely


## trait method resolution

???

## UFCS

```
(Self: Trait<A, B, C>)::method::<D, E, F>()
(int: Sizeof)::sizeof()
(Self: Trait)::method::<D, E, F>(); // calling a method through the type rather than the trait
(Self: Trait)::method::<D, E, F>(&x);
```

static and instance methods have same syntax:
```
trait A {
    // 'static' methods
    fn baz();
    // instance methods
    fn foo(self: &Self); // same as ...
    fn bar(&self); // or 'self' or '&mut self'
    
    // destructuring the 'self' type
    fn method1(MyStruct { a, b, c }: MyStruct);
    
    // methods on smart pointers
    fn method2(ptr: Gc<Self>);
}
```

what about

```
(~Trait:Send: Trait2)::method()
((~Trait : Send) : Trait2)::method() // a? This one
(~Trait : (Send : Trait2))::method() // b?

((G: Graph)::Node : (Trait: Kind))::method() // c?
```

## struct inheritance


### Requirements

 - tree of types (single inheritance)
 - downcasting
 - thin pointers
 - cheap field access
 - easy upcasting
 
### Design

```
// Virtual structs are unsized but *can't* end with an unsized field (so they can be extended)
virtual struct Base { }
impl Base {
    virtual fn foo(...);
    virtual fn bar(...) { }
}

virtual struct Foo: Base { }
impl Foo {
    // Q: How to declare overrides? 'virtual override'?
    virtual override fn foo() { }
}
// Leaves don't need to be virtual (and get static dispatch for virtuals)
struct Baz: Foo { }

```

* base virtual structs don't need to implement virtual methods


## DST

mostly like http://smallcultfollowing.com/babysteps/blog/2014/01/05/dst-take-5/

* 'unsized' keyword
  - does not apply to structs
  - apply to type params `<unsized T: T1 + T2>`
  - unsized types are linear and can be passed as arguments
    - can move out of args, but limitations on where they can be moved *to*
    - can't move into locals
    - can possibly move locals to
    - not into structs, not return
    - think about it harder
* `struct String([u8])`
* "foo" => `&'static String([u8])`
* Naming: "array", Vec, Str, StrBuf
* "unsized T", not "T: Unsized"
* "unsized trait"
  - to check default method implementations in supertraits
* Closure types need to become unsized


# Action items

## RFCs

* GC (felix)
* DST(+ strings) (niko)
* Built-in kinds, static, freeze changes (niko)
* allocators (felix)
* struct fields and privacy (alex)
* UFCS (nrc)
* struct inheritance (nrc)
* intrinsics (brson)

## Other

* open issue on array indexing uints
* alex make logging design changes
* brian make changes to intrinsics
* brian rewrite RFC process
* brian wikifi workweek etherpads
  - emphasize this will produce RFCs
* put dst items in metabug (nrc)


**Note: everything after this is scattered notes and minutes from individual
   topics. It will be mostly unreadable**


# 2014/03/03 DST

Idea: be able to create pointers
```
~[T]
&[T]
Gc<[T]>
Rc<[T]>
```

Also remedy the type grammar inconsistency:

```
T := ~[T]
   | ~T
```

```
U := [T] | Trait
```

```
[T] ==> some number of instances of T, T0, ..., Tn
```

```
[T, ..n] => T0, ..., Tn
```

```
let x: [int, ..3] = [1, 2, 3];
let y: &[int, ..3] = &x;
let z: &[int] = y as &[int]; // Forgot that the length was 3, at least statically
```

```
sizeof(z) = 2 * sizeof(uint)
layout(z) = (ptr, length)
```

```
struct Rc<unsized T> {
    box: *RcBox<T>
} // Rc<...> never unsized

unsized struct RcBox<unsized T> {
    ref_count: uint,
    value: T
} // RcBox<[T]> would be unsized, RcBox<int> not

let x: Rc<[int, ..3]> = Rc::new([1, 2, 3]);
let y: Rc<[int]> = x as Rc<[int]>;
...
let box1: *[int, ..3] = x.box; // thin pointer
let box2: *[int] = box1 as *[int]; // fat pointer (pair with the length)
let y: RC<[int]> = Rc { box: box2 };
```

representation of y:
```
y = RC { box: (ptr, 3) }
```

```
[T] | Trait | Struct<...> 
struct types are unsized if the last field has unsized type
```

```
struct TwoVecs<unsized T, unsized U> {
    x: &T,
    y: &U
}

let x: TwoVecs<[int, ..3], [char, ..5]> = ...;
let y: TwoVecs<[int], [char]> = x as TwoVecs<[int], [char]>;
let z: TwoVecs<[int], [char, ..5]> = x as TwoVecs<[int], [char, ..5]>;
```

```
struct Foo<unsized T> { x: &T }
struct Bar<unsized T> { f: Foo<T> }
Bar<[int, ..3]> => Bar<[int]>
```

```
impl<unsized T> Deref<T> for Rc<T> {
    fn deref<'a>(&'a self) -> &'a T {
        unsafe { &self.box.value }
    }
}
```

- acrichto: seems magical for compiler to bring the length along
- pcwalton: required to make traits work
- niko: simplifies traits/object interaction (will discuss later)
- nrc: tangent: fixed size arrays specify length ([int, ..3]). plans for types to be polymorphic over integers ([int, ...T])?
- niko: someday. think it will work

```
In this scenario, we lose sight of the fact that foo.x and foo.y must have same length, in principle in the future, we could support some sort of capture/existential-open mechanism to track that. 

struct TwoVecs2<unsized T> {
    x: &T, y: &T
}
foo: TwoVecs2<[int]> --> foo.x, foo.y
```

- brson: stil not clear to me that unsafe fat pointers are necessary. makes ffi worse
- niko: i think it is for Deref to be impl'd on Rc
- brson: we'll just need to have lots of lints for ffi
- brson: struct coercion affected by visibility of fields?
- niko: no. i don't think there's any reason Rc type has to care that it was coerced.
- brson: private interior structure determines coercibility - API backcompat hazards
- niko: yes, T can be declared unsized but still not be coercibly
- pcwalton: probably uncommon. you only use this if you want coercion

- acrichto: coercion only works with structs with typarams?
- niko: yep

```
// you want this
struct Tree {
    ...,
    children: [~Tree]
}

// you write this
unsized struct TreeData<unsized T> {
    ...,
    children: T
}

type Tree<T> = TreeData<[~TreeData]>;

let x: Tree<[~Tree, ..2]> = Tree { ..., children: [left, right] };
let y: Tree<[~Tree]> = x as Tree<[~Tree]>;
```

- Where unsized might show up:
  - on type parameters
  - on a trait?? (to reference the `Self` parameter)
  - on a struct (and maybe enum?) itself (ties with opt-in kinds)

```
struct Rc<unsized T> { }
unsized struct RcBox<unsized T>

trait Foo {
}
```

# When can't you use `unsized`:

`<T>`
`<T:Sized>`
`<T:...>`

```
fn foo<unsized T>(x: ~T) {
    let v = *x;    // Bad
    let w = &*x; // OK (maybe fat pointer)
}
```

- Why use a new keyword and not a trait?
- Normally, more bounds mean that you apply to fewer types. This is the inverse.
- But if we did use a trait, consistent of not, there is something appealing:

```
    fn foo<T:Unsized>(...)                  // (1)
    trait Foo : Unsized { ... }               // (2)
    impl Unsized for MyStructType { } // (3)
```

- acrichto: why not some sort of bound? (T: Unsized or T: !Sized)
- dherman: weird that a struct definition is only valid if it's accomanied with "impl Unsized for ..."

```
struct Tree<T:Unsized> {
    last_field: T
}

illegal unless accompanied by

impl Unsized for Tree<T> { ... }
```

- felix: why not have an attribute on a type parameter?
- dherman: attributes makes things look not-so-core
- pcwalton: this is quite a core feature to the language, seems to warrant a keyword

strawman future solution 1:
```
#[declspec(megafoopy)] struct Foo {
    ...
}
```

strawman future solution 2:
```
use megafoopy::megafoopy;
...
megafoopy struct Foo {
    ...
}
```

Not having unsized on Option means:

- `Option<Rc<[T]>>` -- OK
- `Rc<Option<[T]>>` -- Not OK

- acrichto: Can you write `unsized struct` without an `unsized T` typaram?
- nmatsakis: yes, but useless. could add lint for it
- nmatsakis: although, why do we need 'unsized struct'? always redundant
- nmatsakis: what we *really* want is `impl<T:Unsized> Unsized for Foo<T>`, but not using a trait doesn't permit us to write that

# Objects implementing Traits

```
trait ToStr {
    fn to_str(&self) -> ~str;
    fn update(&mut self);
}

~ToStr does not (by default) implement the trait ToStr today

impl ToStr for ~ToStr { ... }

impl ToStr for ToStr {
    fn to_str(&self) { ... self: &ToStr ... }
}

let x: &mut ToStr = ...;
x.update();

let y: &ToStr = ...;
y.to_str();

// the object type Trait implements Trait unless:

// if you use the Self type parameter as something other than the type of the receiver
impl Eq {
    fn eq(&self, bool: &Self);
}
```

```
fn foo<E:Eq>(x: &E, y: &E) { x.eq(y); }

foo::<Eq>(eq1, eq2); // no guarantee that eq1 and eq2 are actually the same underlying type
```

- My proposal:
  - only object-like traits can be coerced to objects
  - object type Trait implements Trait
  - object-like traits are those that do not reference Self and which do not have generic methods

```
trait Map {
    fn insert(&mut self, key: int, value: int);
    fn get(&self, key: int) -> int;
}

// This doesn't work but it's quite reasonable:
impl Map for &Map {
    fn get(&self, key: int) -> int {
        self.get(key); // OK
    }
    fn insert(&mut self, key: int, value: int) {
        self.insert(key, value); // Not ok! 
    }
}

// These are functional but unfortunate:
impl Map for &mut Map { ... }
impl Map for ~Map { ... }
```

- acrichto: qualifications on by-val self?
- niko: 

A potential extension to DST:
```
fn foo(x: [int]) {
    let a = x[0]; // fine
    let b = &x; // fine
    let c = x; // not ok, using x as an rvalue
    bar(x); // fine, passing ownership of x
}

fn bar(x: [int]) {
    ...
}
```

- achrichto: people will get confused about [int] vs &[int]

```
trait FnMut<A,R> {
    fn call(&mut self, arg: A) -> R;
}

&mut FnMut<int, int>    --->    |int| -> int

fn map<F:FnMut<int,int>>(f: &mut F) { ... }

struct Wrapper {
    x: |int| -> int
}

struct Wrapper<F:FnMut<int,int>> {
    x: F
}

fn wrap<F:FnMut<int,int>>(f: F) -> Wrapper<F> { return Wrapper { x: f }; }

|x, y| expr --> new anonymous struct type whose contents are pointers to the closed over variables ("the environment")

|x| expr ==> &mut FnMut<int, int>
|x| expr ==> FnMut<int,int>

trait FnOnce<A, R> {
    fn call(self, arg: A) -> R;
}

fn wrap<F:FnOnce<int,int>>(f: F) { // f: &mut F
    f.call();
}

fn takes_a_closure1(f: FnMut<int,int>) {
    f.call();
    let y = &mut f;
}

fn takes_a_closure2<F:FnMut<int,int>>(f: F) {
    f.call();
}

takes_a_closure(|x| x * 2); // type of |x| is &mut FnMut<int, int>
```

- Summary:
  - unsized keyword
    - `unsized struct`? `Unsized` trait? Just what does this mean, anyhow?
      - Probably the keyword means "unsized if your type parameter is unsized"
    - `unsized trait`? `trait Foo : unsized` ? `trait Foo<unsized Self>`?
- Where can unsized values be used?
  - No assignment to lvalues of unsized type
  - No rvalues of unsized type
    - *Except* parameter passing?
- Objects
  - fn(self)? -- I think this can be supposed, theoretically, independently --ndm
- Strings
  - 

## New decisions

* Where can unsized be rewritten?
  - don't need to make this decision now
* Remove Sized kind
* Rules for which traits can be coerced from objects
  - don't use Self type parameter explicitly
  - don't have generic methods

  
```
trait A {
    fn method<X>(&self, x: &X);
}

trait B { ... }

fn n(a: &A, b: &B) {
    a.method(b); // Error: cannot invoke generic method through object (also the reason below)
}

fn m<T:A>(t: &T, b: &B) {
    t.method::<B>(b); // Error: type parameter X is not declared as unsized, and it is bound to the unsized type B
}

trait A {
    fn method<X>(x: &X);
}

trait B { ... }

fn n(b: &B) {
    A::method(b); //X not unsized
}

```


# DST + strings

niko: strings are just [u8]. natural way to make this work is to have fixed-length strings

Today:

```
"abc"   ----> &'static str
```

Most analogous to vecs:

```
"abc" --> str/3
&"abc" ---> &'static str/3 (coercable to &'static str)
~"abc"  --> ~str/3 (coercable to ~str)
format!("...") -> ~str
```

Alternative pcwalton proposal:

```
"foo" --> String<'static>
~"foo" --> ~String<'static>
pub struct String<'a>(&'a [u8]);
pub struct StringBuffer(Vec<u8>);
format!("abc") => StringBuffer
could rename: String to: Substring, StringView, StringSlice
```

Ideal:

```
"foo" -> &'static String
~"foo" / "foo".to_owned() --> ~String
pub unsized struct String([u8]);
pub unsized struct DOMString([u8]); // weird JS UCS-2 thing

pub unsized struct String<unsized T:ByteArray>(T);
pub unsized struct DOMString<unsized T:ByteArray>(T); // weird JS UCS-2 thing

domstring!("hello")

pub unsized struct InternalString<unsized T> { t: T }
pub type FixedString = InternalString<[u8, ..???]>
pub type String = InternalString<[u8]>

pub unsized struct String<unsized T:StringRepr=[u8]>(T);

String<DOMChar>

pub struct 

pub trait StringRepr /* Encoding */ {
    fn validate(&self) -> ...;
    fn iter<'a>(&'a self) -> Chars<'a>;
    fn len(&self) -> uint;
}

// utf-8
impl StringRepr for [u8] {
}

// ucs-2
impl StringRepr for [DOMChar] {
    ...
}

impl<unsized T: StringRepr> String<T> {
    
    fn foo(&self) {}
    fn repr<'a>(&'a self) -> &'a T { ... }
}

"foo" => &'static String
format!("foo") => ~String
```

- No way to create a `~String` except for format! or creating a buffer 
- `RC<String>` --> 

```
Rc<String> = Rc<String<[u8]>> ... ?? Rc::new("foo") ??

Rc::from_slice("...");

fn from_slice<E:Clone>(s: &[E]) -> Rc<[E]> { ... }
```

- creating a `Rc<[int]>??`

````
let foo: Rc<[int]> = Rc::new([1, 2, 3]);
````

- Issue that arises:
  - cannot create a newtyped [T], essentially, though it seems like it...should work.

ms2ger's DOMstring example:  
https://github.com/Ms2ger/servo/commit/f3653f2615e71a08c5b57e0017637f46387a6d67#diff-69  
ms2ger's concerns:

- lack of auto-slicing
- lack of auto-borrow

#### Fixed length strings:

```
Rc::new("abc") -> Rc<str>
fn from_str(s: &str) -> Rc<str> { ... }
fn from_slice<E:Clone>(s: &[E]) -> Rc<[E]> { ... }

-- sized types:
sizeof::<T>()

-- unsized types:
sizeof::<U>() // not enough information U == [u8]
fn sizeof<T>() -> uint; // Cannot be invoked with an unsized type

let x: &[u8] = ...;
sizeof_value::<U>(x);
fn sizeof_value<unsized U>(x: &U) { ... }

fn from_owned<unsized U>(x: ~U) -> Rc<U> { ... }

fn from_anything<unsized U>(x: U) -> Rc<U> {
    let n = sizeof(&x);
    let box = alloc(n);
    memcpy(&box, &x, n);
    forget(x);
    return box;
}

Rc::new()

Rc::new([1, 2, 3])
Rc::new([1, 2, 3, 4, 5])

Rc::new(*format!("foo, bar, {}", baz));
Rc::new(x.clone())
Rc::new(*(~"foo")) == box(RC) "foo"    ((ish...))

fn new(v: T) -> Rc<T> { ... }

fn new_box<F: FnOnce<T>>(f: F) -> Rc<T> {
    unsafe {
      ... // you must be sure to arrange `rc` gets freed to f()
      move_val_init(*rc.data, f()); // [ acts like *rc.data = f(); ]
    }
    return rc;
}

fn init_with_call<T>(uninitialized_slot: *mut T, f: once proc() -> T) { 
    let box = box();
    scope(failure) { free_uninitialized_box(box) }
    box.data = f();
    return box;
}

fn new_three() -> TempRc<T> { .. }

`~expr` =>
  calls malloc, adds a failure-only cleanup (in case of failure), evaluates expr

Rc::new_box(|| 22); // assuming that `||` yielded a closure
```

- why have box?
  - closures are kinda ugly probably don't want to write them all the time
  - order of evaluation (the allocation of target area versus the computation of the value) matters for quality of LLVM code generation; `box` provides control over this that Rc::new(..) does not.


######

**Wed Mar 05 2014**

```
    unsized struct String {
        priv chars: [u8]
    }
    
    "abc"   has type  &'static String
    
    struct StringBuffer {
        priv chars: *u8,
        priv filled: uint,
        priv capacity: uint,
        ...
        // priv buf: Vec<u8>, ???
    }
    
    impl<E> StringBuffer<E> {
        fn get(self) -> ~String<E>;
    }
    
    impl<E> Deref<String<E>> for StringBuffer<E> {
        fn deref(&'a self) -> &'a String<E> { ... }
    }
```

How do I get an RC<String> (or RC<[u8]> from a Vec)

```
    trait Box<T,P> {
        // `box expr` translates to a call to this:
        fn new<F:FnOnce<T>>(&self, f: F) -> P;
        
        // For sized types, `self.from(&x)` is equivalent `box(self) x.clone()`
        // For an unsized type, `x.clone()` would be an error because it would have type `[u8]`.
        fn from<unsized T:CloneTo>(&self, value: &T) -> P;
    }
```

```
// Writes a copy of `Self` into `*p`.
//
// Callee may assume that `*p` is writable and
// has enough memory for `sizeof_value(self)`.
// This is used by smart pointers to clone
// unsized values.
pub unsized trait CloneTo<T> {
  unsafe fn clone_to(&self, out_pointer: *mut Self);
}

// Implementation for sized types:
impl<T:Clone> CloneTo for T {
  unsafe fn clone_to(&self, p: *mut T) {
      *p = self.clone();
  }
}

impl<unsized T:CloneTo> CloneTo for [T] {
  unsafe fn clone_to(&self, p: *mut [T]) {
      assert_eq!(self.len(), len(p)); // should be able to do this
      for i in range(0, self.len()) {
          self[i].clone_to(&(*q)[i]);
      }
  }
}
```

```
fn foo(x: [u8]) {
    print(x[0]); // prints i
    bar(x);
    print(x[0]); // prints i+1 // ERROR
}

fn bar(mut x: [u8]) { // ERROR
    x[0] += 1;
}

Rc::new(*"abc"); // ERROR

static foo: [u8, ..3] = [1, 2, 3];
bar(*&foo); // foo is in immutable memory
```

```
impl<A> Box for RcIn<A> {
    fn new<unsized T>(&self, t: T) -> Rc<T> { // aka "move_from"
    }
    
    fn clone_from<unsized T:CloneTo>(&self, t: &T) -> Rc<T> {
        unsafe {
            let size = sizeof_value(t);
            let p = self.alloc(...);
            t.clone_to(p.value);
            return Rc(t); //...
        }
    }
}

let foo: Vec<File> = ...;
let bar: Rc<[File]> = Rc::new(*foo.get()); // get() ==> ~[File]
```

So to use this monstrosity:

```
let x: Rc<String> = Rc::from("abc"); // "abc" has type &String

let b = StringBuilder::new();
b.push("Hello, ");
b.push("World");
let y: Rc<String> = Rc::from(b.to_slice()); // b.to_slice() has type &String

let x: Rc<&'static String> = box(RC) "foo";

let foo = "bar";
```

#### acrichto's proposal

`"bar"` has type `String` and corresponds to a freshly generated stack slot with the string (logically, at least)

`&"bar"` yields `&'static String`
    (have to be careful)
    (`&[1, 2, 3, 4, 5, 6]`)

`Heap::new("bar")` yields `~String`
`Rc::new("bar")` yields `Rc<String>`

`box "bar"` yields error and suggests using `Heap::new`

`let foo: Vec<File> = ...;  
let bar: Rc<[File]> = Rc::new(*foo.get()); // get() ==> ~[File]`





## Decisions

* Naming: "array", Vec, Str, StrBuf
* "unsized T", not "T: Unsized"
* "unsized struct", "unsized trait"

## Action Items

* Closure types need to become unsized

## Work plan

* convert `~[T]` -> `Vec<T>`
* add unsized keyword and compute whether type is sized
* remove Sized trait
* check all exprs w/ unsized type for validity
* add `[T]` type ( `{T}`?) and implement coercion rules from `[T, .. N]` -> `[T]`
  - add fat pointer notion
* indexing of `&[T]` in trans
  - extracting length and bounds checks
* switch `&[T]` to `&([T])`
* structs with unsized final field propagate fat value
* reimpl strings
* switch traits



# 2014-03-03 static/const

- Is the address significant?

```
static foo: int = 3;
```

- What kinds of values can go in there

## Current rules, see https://github.com/mozilla/rust/pull/11979
   * For each mutable static item, check that the type:  
     * cannot own any value whose type has a dtor
     * cannot own any values whose type is an owned pointer
   * For each immutable static item, check that the value:  
     * does not contain any ~ or box expressions (including ~[1, 2, 3] sort of things, for now)
     * does not contain a struct literal or call to an enum variant / struct constructor where  
       * the type of the struct/enum is freeze
       * the type of the struct/enum has a dtor

```
static foo: Option<~int> = None;
```

- Interior mutability and Freeze

```
&T -- immutable reference but that's a lie
aliasable reference
for most types T, aliasability implies immutabity
Cell, Atomic
```

```
static NATIVE_MUTEX_INIT: NativeMutex = ...; // this is in error (NativeMutex not Freeze)
...
static mut INIT_LOCK: NativeMutex = NATIVE_MUTEX_INIT;
```

```
&NativeMutex -- 
(&NATIVE_MUTEX_INIT).lock(); // Crash because this would be mutating read-only memory
```

- but we don't crash because value must be Freeze hence NATIVE_MUTEX_INIT gives an error

- Cross-crate questions -- do I have to recompile a downstream create if I change the initial value of my static? or my const?

```
const T: Type = ...;
// Rules are the same as for static but Freeze is not required
// You can't take the address of a const, directly or indirectly
```

```
const NATIVE_MUTEX_INIT: NativeMutex = ...; // this is ok
...
static mut INIT_LOCK: NativeMutex = NATIVE_MUTEX_INIT;
```

- const would be classified as an RVALUE, like `None` (I guess)
- static would be classified as an LVALUE, aliasable

```
const MyConst: Type = SomeInitializer;
fn MyConst1() -> Type { SomeInitializer }

let x = MyConst;
let y = MyConst1();
static z: Type = MyConst;
```

```
enum Option<T> { Some(T), None }
```

```
let x = &None; // allocates stack space and initializes it with None
let x = &SOME_STATIC_VARIABLE; // does not allocate stack space
```

- Significant addresses:
  - used by TLS
  - format! strings are explicitly INSIGNIFICANT
    - Might be that tying const/static together is wrong
- Disallow taking address of a static unless you flagged it with significant address
- Maybe just disallow taking address of a static unless it is Freeze
- Significant addresses:
  - No for static, unless tagged with 
  - Yes for static mut

#### Resolution

- Disallow taking the address of a non-mut static value unless it is Freeze
- Significant address policy is:
  - if static mut, yes
  - if static, no, unless tagged with some attribute or other
- Permit non-freeze types in statics

#### Second possibility

- Remove static mut altogether
- If a static is not Freeze, stored in mutable memory, address is significant
- If a static is not Share, cannot take its address
- Cannot create instance of non-Freeze struct unless it is Share (in a static initializer)
- Introduce const, which is an rvalue, including support for generics

- Pro: does not admit unsafe code
- Con: const and static

#### Third possibility

- Disallow taking the address of a non-mut static value unless it is Freeze
- Permit non-freeze types in statics
- Safe to create `&` pointer to static mut iff data is Share (maybe other cases?)
- Unsafe to read out from static (except maybe in some cases?)
- Unsafe to assign to static mut
- Unsafe to create `&mut` pointer to static mut


http://ryanarn.blogspot.com/2011/07/curious-case-of-pthreadatfork-on.html


# 2014/03/04 GC


#### "THE CHART"

```
                     |MRK|MOV|GEN|INC|
Exact rooting        |   |   |   |   |
Write barrier        |   |   |   |   |
Read barrier         |   |   |   |   |


✓
MRK = Mark/Sweep
MOV = Moving
GEN = Generational
INC = Incremental

```

Axes:
- How much do we know?
  - Fully conservative
  - Fully precise, incomplete (identify all roots but not ptrs)
  - Fully precise, complete (identify all roots and ptrs)
  - Mostly copying (reachable from roots -> pinned)

- How do we identify the root set?
  - `~T`
  - Smart pointers
    - Do we have to register memory?

- How we trace heap objects?
  - For Cheney algorithm, we have to know what to pin before we start
  
- How to define custom tracing?
  - Attributes on fields
  - Traits and user-defined hooks
    - Exposes the GC machinery to some extent
    - What happens if you fail to implement the trait?

- User-defined code
  - Destructors and finalization
  - Trace hooks

#### USE CASES / FEATURES

- User-defined malloc storing GC pointers
- What can go into a GC?
- Fully participating in the trace / reloc. cycle
- What kinds of restrictions on destructors, finalizers?
- Supporting shared state reachable via GC'd data
- Finalizers

####

- felix: GC abstraction for storage. objects in shared GC heap being managed. Things outside that space are often thought of as roots.

(whiteboard)

- felix: Goal is to find connected components that include roots, specifically identify the (approximately) smallest connected component.
- felix: how? tracing is a fixed-point algorithm. three sets: 1) unknown state, 2) scheduled to be scanned (enqueued in work set) 3) things that have been scanned. 2 + 3 are marked as live or dead.
- felix: fixed point: start with everything unknown, enque everything reachable from roots, scan everything in the work set to find what they point to.
- felix: main impls: mark/sweep - objects don't move in memory, mark bit exists somewhere, flag it when enqueued to be scanned;
- felix: copying - every time you encounter object, you make a copy and install a forwarding pointer
- felix: with mark/sweep you have a bark bit; with cheney copying collector. (details)

- nrc: can you discuss adv/disadv of both? mark/sweep doesn't require copy
- felix: not doing copy is adv, but by doing copy you get compaction, reducing fragmentation
- nrc: js guys do huge amount of work to go from mark/sweep -> moving. why?
- felix: frag is one. nurserey can be fixed area of memory which makes it easy to determine which objects are in the nursery, makes write barriers easier
- nrc: why write barriers?
- felix: in generational you need to be able to identifiy pointers from outside of nursery to inside (remembered set)
- pcwalton: another: if you are doing copying allocation then nursery allocator can be a bump allocator
- nrc: can mark/sweep do generational?
- felix: yes
- pcwalton: boehm does it

- dherman: what's the prescription for rust?
- felix: want to leave door open to either in the future

## Issues for Rust

- felix: choice: fully conservative vs. fully precise, vs. mostly-precise/mostly-copying.
- felix: fully conservative: look at all words and guess whether they are pointers. won't find true minimum connected components. scan stack and objects conservatively.
- felix: fully precise: precise knowledge of roots in the stack and every object. llvm can't do this now, which is fine. mostly-copying is good enough for us
- felix: mostly-copying: roots conservatively scanned, objects precisely. any object that is only reachable via heap can be moved

- brson: why move but not use a generational algorithm? write barriers
- felix: compaction
- felix: two kinds of tracing in Rust: 1) things that are conceptually roots 2) things that are conceptually objects
- felix: if we have user-defined smart pointers and we want to put references to GC objects. As long as smart pointers live on the stack then we can conservatively scan them. Problem is that smart pointers don't necessarily live on the stack.
- dherman: don't we need trace hooks? then the problem is 'are they tied to one algorithm'?
- felix: do we let users define trace hooks?
- dherman: must be manual since Rc's are unsafe code
- felix: 2 issues: tracing of smart pointers you find 'somehow'; registering allocated memory with GC.
- dherman: if they are traits and we discover someday they don't work we can add new traits for new strategies
- felix: q's: how can traits be used to future proof?
- felix: reason to distinguishe between 'tracing roots' and 'tracing objects', if we just had a single API to expand the root set, problem that in cheney mostly-copying, everything conservatively pinned must be done at outset. in the case where there's a GC object that has only ref to smart-pointer, if we don't consider it a root

- felix: risk of user-defined code in trace hooks: similar to finalizers
- felix: with tracehooks we can limit what code does. finalizers can do *anything*
- felix: can imagine limiting what sort of data is allowed in GC pointers
- felix: i don't want to duplicate Java/C-sharp...
- dherman: sounds like you are talking about something like dtors but not. i'm not sure what restrictions we have on dtors. the ability to refer to yourself.
- niko: have very restrictive dtor rules currently, particularly that you have to have owned data. limiting for raii specifically, may not be relevant because in RAII case you always have a borrowed pointer.

- felix: dtors vs. finalizers. dtors run at a time prescribed by their scope. finalizers run at unpredictable times. can't reason about them statically.
- dherman: where do you define finalizers? what kind of data structures?
- felix: not saying we want a 'finalizer', but we need a way to do resource cleanup.
- felix: guardians: GC doesn't call finalizers, types with finalizers are registered with guardians. guardian is an actual object that the mutator has a referenc to.
- felix: guardian has a queue of associated objects that are dead
- felix: mutator is responsible for running finalizers


## A sort of big topic that Felix wanted to talk about: how is tracing driven?

- felix: dynamic type driven: record the type in the object or in a table
- felix: static: with precise stack scanning you can get the types from the roots
- felix: not sure how different these options are in practice
- felix: tricky because now we don't pass type descriptor to allocators

- niko: how do we find the root set? we can scan the stack. 2 scenarios i'm concerned about: smart pointers on the stack we need to trace; floating roots
- felix: root set = ~T and transitively all other ~T; other kinds of smart pointers (Rc<T>)
- alex: how do you know a stack slot is ~T?
- felix: have to register ~ that could contain GC roots.
- niko: you could track ~T with stack maps and not dynamically


# 2014/03/4 Allocators

```
trait Alloc<E, P> {
      fn new(&self, value: ||->E) -> P
}
 
structu Rc<T, A=Heap> { ... }
```
 
- nmatsakis: Motivation: we want to be able to paramaterize data structures over the allocation scheme that they're using to allocate their memory. Then, you can instrument and measure. Might want to redirect to the JS allocator in Servo; might want to instrument memory allocations.
 
- pcwalton: Segregated fit is extremely common. It's used in Gecko for the frame tree, which knows how to allocate, using segregated fit (fixed-sized allocation bucket pools). It's one of the most common use cases for allocators.
 
- nmatsakis: Idea is that you would call Alloc::new to initialize the memory. E is the type of object that's in there, and P is the pointer that results. These would be linked; P would be a wrapper (e.g. smart pointer) around E that manages it. An example of such an allocator is for ~ in our standard library.
 
```
impl <T> Alloc<T, ~T> for Heap {
    fn new(&self, value : ||->T) => ~T {
         ~value()
    }
    }
```
 
- nmatsakis: Rc doesn't care where it gets its memory; only that it gets some. So, it might be:
```
struct Rc<T, A=Heap> {}
```
- nmatsakis: Note that Heap does not need a <T> here. 
```
impl<T, A> Alloc<T, Rc<T>> {
  fn new(...) {
    let ptr = self.alloc.new(|| RcBox{ rc: 1, value: value());
    ptr
 }
}
```
 
- nmatsakis: That's what we laid out before. So, you can build up chains of these from there.
- acrichto: Allocator per smart pointer?
- nmatsakis: If you use Heap, it's a singleton type, and just falls out.
- pcwalton: Two important things here. 1) allocators must not be specialized to a single type and must be able to allocate many different types. 
- nmatsakis: You could write such an allocator!
- pcwalton: But any API we come up with must be able to allocate multiple types into the same thing. Users must be able to define such allocators themselves. That's required for b-trees because you might have to allocate tuples of the objects. 2) Some allocators may have custom instance data.
- brson: IF you need back pointers to the allocator, don't you need a stack discipline for the allocator? or Gc on it?
- pcwalton: Or a reference count to the allocator. There's no way around it; fundamental limitation. I think most people will have an allocator that lives forever.
- nmatsakis: I don't think there are back pointers in this design except for destructors and where the call free on the allocator. 
```
struct HashMap<K, V, A=Heap> {
      buckets: Vec<Option<(K, V)>, A>
}
```
- acrichto: Seems backwards to require an allocator for every smart pointer. You might have the heap and the gc heap, possibly...
- nmatsakis: We're hoping to unify the box keyword. You give it a value and it allocates memory within that allocator with that value.
- acrichto: If you have an allocator for some heap, how do you allocate a bunch of different types on it?
- pcwalton: Eventually, the semantics just come down to - when do you call free?
- acrichto: But, if I have an allocator and want to allocate some Rc things, a hashmap, etc. how do I do that? So the allocator has to implement clone...
- pcwalton: They have to be reference counted. I suspect games have a fixed set of bins (enemies, etc.) and they're each fixed-sized and live forever. Everything is `&'static`
- brson: What about heavily-typed allocators?
- pcwalton: Question is not dealing with typed vs. untyped. The mistake of the STL allocators is that an allocator in it can only allocate an object of one type. That's not the case in this. Or maybe it's multiple, but you have to supply them all up front. You really want to be able to use an allocator separately from the set of types.
- dherman: Is this a perf or programming problem?
- pcwalton: This is a programming problem (though there are perf problems, too, related to virtual function calls). The question we have is do we want to unify the unique pointer and the interface you use with the allocator. In this proposal, we hand back a unique pointer and the destructor. In the EASTL, you need a wrapper because there are no safe lifetime semantics, requiring some kind of unique pointer wrapper. With niko's, you have the allocator itself providing a more safe interface, though the interface is slightly more complicated.
- pcwalton: If you did the EASTL approach in Rust, you would need a thing like a unique pointer (or ~). Then it just gives you back an object that you don't manage explicitly, it does.
- huon: How do vectors work? Want to make one of size 100...
- pcwalton: They are just another safe wrapper around the allocator in the EASTL appraoch. I think it would be a separate method....
- nmatsakis: Might want a separate one around uninitialized memory. Vecs are one example; hashmaps are another. Often want to reserve but not fill in the memory yet.
- pcwalton: EASTL always gives you uninitialized memory.
- nmatsakis: That's not very Rust-y. box operator would not do it. box should just be sugar like:
```
box(alloc) expr ==> alloc.new(|| expr)
```
- nmatsakis: I was thinking that the allocator would give you a *T that is uninitalized...
- pcwalton: Probably don't want sized & aligned (can get from intrinsics).
- brson: The EASTL alignment requirements are much more so than that.
- pcwalton: Use newtype. Don't want people passing alignment information down to Rc. Rc::with_alignment is bad.
- pcwalton: I like separating ownership questions from allocation questions. Our two proposals agree on those fronts; mainly just aesthetic issues. Hrm, maybe EASTL makes it easier to allocate *T. 
- brson: What's the back pointer?
- pcwalton: All of the smart pointers have one.
- brson: So ~ does? Every ~ has a back pointer to the allocator? So ~ is now two words?
- pnkfelix: In practice, usually zero-sized. 
- pcwalton: Not a pointer to the allocator; copy of the allocator! So that can be of size zero. 
- brson: One thing the EA people complained about was copying the allocators in STL. I don't know how they could work around it.
- pnkfelix: Is there a way to allocate an array of some elmeents with only a single clone? Where in API?
- pcwalton: Wrapper for vec just has it. Responsibility of the smart pointer. Probably just falls out of ~ and DST. Because all the elements are one "thing" as far as DST is concerned anyway.
- pnkfelix: So it'd be laid out as (allocator, elements)?
- pcwalton: No. (ptr, allocator) where ptr is to the elements. Might be able to collapse it.
- huon: isn't putting the allocator behind the pointer like putting the box headers on everything?
- pnkfelix: Difficulties with headers were unsafe code... and inefficiency?
- huon: Biggest was differences in representation between @ pointers and non-@.
- pcwalton: It's going back to having headers if you have a non-zero size headers.
- nmatsakis: Used to use different layouts depending on if T was managed or not, which was a real pain in trans (and it was in the language). The default allocations would just be a straight call to malloc. Just when using custom allocators that things look different.
- pcwalton: Generalized (malloc/free). Fixed-sized ones with common size/alignment. Small block allocators just do those. Stack allocators are just bumping (users can't free). Page-protected trigger h/w exceptions when writing out of bounds. Non-local deal with memory that's slow to read with stuff in system memory separate from the data, which is elsewhere. Handles allow compactions; reference through the allocator. Page allocators use system allocators like mmap.
- larsberg: COM used fixed block sizes and always allocated certain word-sized chunks, with the size of the chunk as a header, etc.
- pcwalton: Similar to JS.
- nmatsakis: 
 
```
trait Alloc<T> : Clone {
    // Failure semantics?
    fn malloc(&self) -> *T;
    fn malloc_many(&self, n: uint) -> *T;
    
    // We will want to noodle around with the precise set
    // of free operations due to DST.
    fn free(&self, ptr: *T);
}
 
trait Box<T,P> { // "Smaht"
    fn box(&self, value: once || -> T) -> P;
}
 
// "System heap": libc malloc and free
struct DefaultAlloc;
impl<T> Alloc<T> for DefaultAlloc {
    fn malloc(&self) -> *T { ... }
    fn free(&self, ptr: *T) { ... }
}
 
// fixed type allocator
struct FixedTypeAllocator<T> { .. }
impl<T> Alloc<T> for FixedTypeAllocator<T> {
}
 
// arena allocator
struct MyArena { ... }
struct Arena<'a> { theArena: &'a MyArena }
impl<'arena,T> Alloc<T> for Arena<'arena> {
    ...
}
 
// Owned pointers
// box(OWNED) 22
// box(OwnedIn(myAllocator)) 22
struct Owned<T,A=DefaultAlloc> { box: *T, alloc: A }
struct OwnedIn<A>(A);
static OWNED = OwnedIn(DefaultAlloc);
impl<T,A: Alloc<T>> Box<T,Owned<T,A>> for OwnedIn<A> {
    fn box(&self, value: once || -> T) -> Owned<T,A> {
        let ptr: *T = self.alloc.malloc();
        // Cleanup: ptr
        ?? *ptr = value();
        Heap { box: ptr, alloc: self.alloc.clone() }
    }
}
impl<T,A: Alloc<T>> Drop for Owned<T,A> {
    fn drop(&mut self) {
        self.alloc.free(self.box);
    }
}
 
// Rc pointers
// box(RC) 22
// box(RcIn(myAlloc)) 22
struct Rc<T,A=DefaultAlloc>> { box: *RcBox<T,A> }
struct RcBox<T,A> {
    alloc: A,
    ref_count: Cell<uint>,
    value: T
}
struct RcIn<A>(A);
static RC: RcIn<DefaultAlloc> = RcIn(DefaultAlloc);
impl<T,A: Alloc<RcBox<T, A>>> Box<T,Rc<T,A>> for RcIn<A> {
    fn box(&self, value: once || -> T) -> Rc<T,A> {
        let ptr: *RcBox<T,A> = self.alloc.malloc();
        // Missing: cleanup ptr
        ptr.alloc = self.alloc.clone();
        ptr.refcount = Cell(1);
        ptr.value = value();
        Rc { box: ptr }
    }
}
 
// Containers
struct Vec<T,A> {
    data: *T
}
 
 
```
 
 
- Not everything is compatible.
- If you had a fixed type allocator for uints, you couldn't do `box(RcIn(fixedAlloc)) 22`
- With HKT, you should be able to avoid leaking all details
 
Using the arena allocator
```
let arena: Arena<'a> = ...;
let data: Owned<uint,Arena<'a>> = box(OwnedIn(arena)) 22;
 
impl<'a, T> Box<T, &'a T> for Arena<'a> {
    fn box(&self, value: once || -> T) -> &'a T {
        let memory = self.theArena.alloc_slot();
        *memory = value();
        return transmute(memory);
    }
}
```
 
# EASTL
 
- Throw
- 
 
# NDM -- things to consider
 
- DST: Allocators must be aware that alloc can be called with [T, ..5] and then converted to [T]?
- GC: Can GC trace through all allocated memory? No -- might be uninitialized!
 
# Where do we need generics over allocators?
 
- Data structures: yes
- Sort (temporary arrays): no, probably not, leaks data
- Higher-level APIs: no?

# 2014/03/4 Built-in traits

I = shallow (interior)
D = deep

* Freeze (I) - &T: T shallowly immutable
* Share (D) - &T: T deeply threadsafe, including primitives, &mut T
* Send (D) - T is (('static && Share) || deeply owned) and logically transferrable, doesn't imply Share
* Pod (I) - non-linear types that can be safely memcpied, including &T, and *T
* Sized (I) - contains only types with statically known size (also Sized)

action items:
* rename Pod to Copy
* add Share
* remove Freeze
* Add Unsafe<T> (initially proposed as Mut<T>)

```
// Cannot be placed in statics at all, may dictate optimizations
#[lang = "unsafe"]
struct Unsafe<T> { priv value: T }
impl<T> Unsafe<T> {
    fn new(t: T) -> Unsafe<T> { Unsafe { value: t } }
    unsafe fn get(&self) -> *mut T { cast::transmute(&self.value) }
}
```



# 2014/03/05 Opt-in builtin traits

* Send - transferrable between tasks
* Share - &T deeply threadsafe
* Copy - POD

Freeze no longer exists because...

```
impl Unsafe<T> {
    fn get(&self) -> *mut T;
}
```

**For statics, references cannot be any type that contains `Unsafe`**

- brson: Unsafe<T> is opt-out unlike the other traits
- felix: You can always have Unsafe<()>
- niko: reserving the right to be unsafe in the future
- brson: doesn't this seem bad?
- niko: this is only for statics, and perhaps some invisible analyses
- all: ok, sounds good

- niko: All except Sized require an impl or deriving.
- niko: unsafe pointers have all these kinds, but before it was unlikely your API fulfulled all the kinds requirements.

- acrichto: lets see example of Mutex.  Make sure we can't do `Mutex<RC<U>>`.
- niko: Why again is `Mutex<RC>` bad?
- acrichto: because then someone can clone() the rc and obtain non-mutex-guarded access
- niko: (whiteboard prototyping:)

```
impl <T:Send> Send for Mutex<T> { }
impl <T: Send || Share> Share for Mutex<T> { } // <-- `||` not yet legal syntax
// the latter might alternatively be expressed by loosening coherence for empty impls
// and doing:
impl <T:Send> Share for Mutex<T> { }
impl <T:Share> Share for Mutex<T> { }
```

Important counter examples:

- Why not unify `Send` / `Share` => `Cell<T>`
- `Mutex<T>` wants `Send || Shar`e because `Mutex<Cell<T>>` is ok, `and Mutex<&mut T>` is ok (wacky)
- `RwArc<T>` wants `Share` but not `Send`, because you can get multiple readers
- but maybe `Mutex<T>` can just take `Send` because you already have `RwArc` for `Share`


## deriving(Data)

Use the Eq, Ord, Clone, Hash, Send, Share

Let's not decide on deriving(Data)

## Unsized bound

```
trait CloneTo: Unsized {
    
}

impl<T: Unsized+CloneTo> 
```


## Decisions

* Mutex<T: Send> - Share types use RWArc
* No decision on deriving data


# 2014/03/05 - struct fields + priv

Analysis: https://docs.google.com/spreadsheet/ccc?key=0AqUaVEtntL3PdC1QVEVCQ0t0QzFMelF3VUc3NzEzblE#gid=0

```
pub struct Point {
    pub x: int,
    pub y: int,
}

pub struct Point {
    pub:
    x: int,
    y: int,
}

pub struct Point pub: {
    x: int,
    y: int,
}

pub struct Point { pub:
    x: int,
    y: int,
}

pub struct Point {
pub:
    x: int,
    y: int,
}

pub struct Point {
pub: x: int,
       y: int,
}

pub* struct Point {
    x: int,
    y: int,
}

struct Point { x: int, y: int }
class PointObject { x: int, y: int }

```

```
pub struct Foo(pub Bar, Baz, pub Quux);
```

statistics: https://gist.github.com/9376568

* Felix's wacky pattern-matching desire with all pub tuple structs can be handled by single-variant enums.

## Decisions

* switch fields to priv by default
* remove private enum variants
* convert 'priv' to a reserved word
* tuple struct fields are private by default
* tuple struct fields take the pub keyword
* enum variant struct fields have to stay public
* identify structs that want to be priv tuple-structs and change them




## SERVO RESULTS

```
Structs  # Private fields  # Public fields  # strustc all private fields  # structs all public fields

alert
        2       0       3       0       2
azure
        27      4       62      4       23
css
        26      14      50      6       20
egl
ERROR
encoding
        28      4       48      2       26
geom
        12      0       64      0       12
glfw
        11      2       34      2       9
glut
ERROR
        1       2       0       1       0
harfbuzz
        0       0       0       0       0
http
ERROR
hubbub
        15      0       77      0       15
layers
        54      40      100     10      42
libparserutils
nss
opengles
        6       6       0       6       0
png
        2       2       4       1       1
sharegl
        14      146     8       10      4
skia
spidermonkey
        31      0       126     0       31
stb-image
        2       3       4       1       1
android
linux
ERROR
        201     0       1465    0       201
macos
        50      26      73      24      26

script
        89      144     209     38      51
util
        40      74      56      22      16
```

### Servo itself
```
[larsberg@lbergstrom components]$ ~/tmp/analyze util/util.rs 
        20      37      28      11      8
[larsberg@lbergstrom components]$ ~/tmp/analyze gfx/gfx.rs 
        56      34      137     13      40
[larsberg@lbergstrom components]$ ~/tmp/analyze msg/msg.rs 
        3       0       8       0       3
[larsberg@lbergstrom components]$ ~/tmp/analyze main/servo.rs 
        85      96      204     42      41
[larsberg@lbergstrom components]$ ~/tmp/analyze net/net.rs 
        8       14      12      3       5
[larsberg@lbergstrom components]$ ~/tmp/analyze style/style.rs 
        44      72      99      19      25
[larsberg@lbergstrom components]$ ~/tmp/analyze script/script.rs 
        146     25      315     14      130
[larsberg@lbergstrom components]$ 
```

### Servo platform components

```
[larsberg@lbergstrom linux]$ ~/tmp/analyze rust-fontconfig/fontconfig.rc
        6       0       17      0       6
[larsberg@lbergstrom linux]$ ~/tmp/analyze rust-freetype/freetype.rc 
        27      0       198     0       27
[larsberg@lbergstrom linux]$ ~/tmp/analyze rust-xlib/xlib.rs 
        84      0       625     0       84
[larsberg@lbergstrom linux]$ 

[larsberg@lbergstrom macos]$ ~/tmp/analyze rust-core-foundation/lib.rs
        15      11      24      10      5
[larsberg@lbergstrom macos]$ ~/tmp/analyze rust-core-graphics/lib.rs 
        5       0       8       0       5
[larsberg@lbergstrom macos]$ ~/tmp/analyze rust-core-text/lib.rs 
        3       2       1       2       1
[larsberg@lbergstrom macos]$ ~/tmp/analyze rust-io-surface/lib.rs 
        1       0       1       0       1
```
     
### Servo support components

```
alert
        2       0       3       0       2
azure
        27      4       62      4       23
css
        26      14      50      6       20
egl
        118     0       736     0       118
encoding
        28      4       48      2       26
geom
        12      0       64      0       12
glfw
        11      13      36      1       10
glut
        1       2       0       1       0
harfbuzz
        0       0       0       0       0
http
        11      5       38      0       8
hubbub
        15      0       77      0       15
layers
        60      50      100     16      42
libparserutils
nss
opengles
        6       6       0       6       0
png
        2       2       4       1       1
sharegl
        14      146     8       10      4
skia
spidermonkey
        31      0       126     0       31
stb-image
        2       3       4       1       1
```

# 2014/03/05 small topics

### integer sizes

#### Problems

* platform specific sizes for default ints has overflow hazards
* performance of word-size ints isn't best on all platforms

#### Decisions

* array indexing needs to be strictly uints
* leave shifting taking any integer types

### well-behaved iterators

* do nothing

### logging

problems:

* requires ugly crate map

acrichto: if we get rid of logging's dep on crate maps we can get rid of crate maps

#### decisions

* two variables affect logging, the global log level, and the local path
* global log level is a static mut in std, initialized to 255
* log!(...) emits code like the following

```
macro_rules! log(
    ($lvl:expr, $($arg:tt)+) => ({
        let lvl = $lvl;
        if lvl <= ::logging::GLOBAL_LOG_LEVEL {
            if ::logging::logging_enabled(my_module_string!()) {
                format_args!(|args| {
                    ::logging::log(lvl, args)
                }, $($arg)+)
            }
        }
    })
)


// logging.rs
pub static mut GLOBAL_LOG_LEVEL: uint = MAX_LOG_LEVEL;
static mut GLOBAL_LOG_MAP: *mut HashMap = 0 as *mut HashMap;
fn logging_enabled(lvl: uint, s: &str) -> bool {
    static INIT: Once = ONCE_INIT;
    INIT.doit(|| {
        GLOBAL_LOG_LEVEL = parse_log_level(getenv("RUST_LOG"));
        GLOBAL_LOG_MAP = parse_log_map(getenv("RUST_LOG_PATHS"));
        rt::at_exit(proc() { free(GLOBAL_LOG_MAP) });
    })
    
    if lvl > GLOBAL_LOG_LEVEL { return }
    global_map_contains(s)
}

local_data_key!(local_logger: ~Logger);
pub trait Logger {
    fn log(&mut self, lvl: uint, s: &str, arg: &fmt::Arguments);
}

````

* logging is initialized *lazily* the first time the log level is checked
* change logging spec to be more like
* remove __log_level and all logging machinery

### semicolons and tail exprs

```
macro_rules! foo( () => (1) )

fn bar() -> int {
    if true { 1 } else { 2 } - 1 // this is an error today
}

fn main() {
    println!("{}", bar())
}
```

### intrinsics

* don't allow taking value of intrinsics
  - so we can do special type checking on transmute
* transmute gets type checked specially
  - both types must be known to have the same size
  - either is a non-pointer to generic then no
  - both must be sized or the same type
  - niko would not object to a warning about alignment
* reexport transmute, init, uninit, move_val_init, offset intrinsics instead of using wrappers
* cleanup rustdoc to indicate intrinsics nicely




# 2014/03/06 Trait method resolution

```

// Add : (Self, RHS, SUM)
trait Add<RHS, SUM> {
    fn add(lhs: &Self, rhs: &RHS) -> SUM;
}

impl Add<i32, i32> for i32 { ... }
impl Add<i64, i64> for i32 { ... }
impl Add<i64, f64> for i32 { ... }

impl Add<Vector, Vector> for Vector { ... }
impl Add<Scalar, Vector> for Vector { ... }
```

- Today:
  - we only consider the `Self` type (more or less); RHS and SUM are then implied by it
  - `Self -> RHS, SUM`
  - `(Self, RHS) -> SUM`

- Methods
  - `a.foo(...)`
  - `a.foo(b)`
  - I think we should only have to know `a`
  - We generally don't allow overloading
  - Plus we take advantage of top-down inference:
      - `a.foo(..., |x, y| x + y)` 
      - knowing what method `foo()` is being called, we can (usually) know types of `x` and `y`

- Coherence:
  - We could distinguish methods from fns
  - Impose stricter rules if a method notation is impose
  - Impose 

- 





```
// crate A

pub trait Foo {}

// crate B
extern crate a;

pub trait Bar {}
impl<T> Bar for T {}
impl<T: Bar> a::Foo for T {} // ERROR: Fails Orphan check
//// (fails orphan check because in general neither `T` nor `a::Foo` is defined by crate B)

// crate C

extern crate a;
struct Int { }
impl a::Foo for Int { }

// crate D

extern crate b;
extern crate c;
```



## Questions

1. do we need any sort of fundeps to preserve some important consistency properties?




# 2014/03/6 UFCS

problems:

```

fn f<Y: T1, X: T1 + T2>(x, y) {
    T1::method(); // does this call X's impl or Y's impl
    x.method2() // is it T1.method or T2.method?
}
```

### Proposal 1

- nrc: UFCS uses '.' for both static and instance methods. 

```
X.T1::method();
x.T1.method2();
```

- problem: ambiguity "x.method()" is "x" a type or is "x" a method

```
fn foo<x: x>(x: x) {
    x.method() // what's happening here?
}

// full syntax
X.T::<T1, T2, T3>::method::<T4, T5, T6>() 
```

### Proposal 2

```
Trait::<A, B, C, for Self>::method::<D, E, F>()
Sizeof::<for int>::sizeof(); // a little odd
Self::method::<D, E, F>(); // calling a method through the type rather than the trait
x.method::<D, E, F>();
```

- brson: this is only associated functions?
- niko: you promote methods to associated functions (UFCS)

```
int::sizeof(); // under proposal 2
int.sizeof(); // under proposal 1

X.T1::<A, B, C>::method::<D, E, F>(); // fully qualified version 1
T1::<A, B, C, for X>::method::<D, E, F>(); // fully qualified version 2
```




## Decisions

```
(Self: Trait<A, B, C>)::method::<D, E, F>()
(int: Sizeof)::sizeof()
(Self: Trait)::method::<D, E, F>(); // calling a method through the type rather than the trait
(Self: Trait)::method::<D, E, F>(&x);
```

static and instance methods have same syntax:
```
trait A {
    // 'static' methods
    fn baz();
    // instance methods
    fn foo(self: &Self); // same as ...
    fn bar(&self); // or 'self' or '&mut self'
    
    // destructuring the 'self' type
    fn method1(MyStruct { a, b, c }: MyStruct);
    
    // methods on smart pointers
    fn method2(ptr: Gc<Self>);
}
```

what about

```
(~Trait:Send: Trait2)::method()
((~Trait : Send) : Trait2)::method() // a? This one
(~Trait : (Send : Trait2))::method() // b?

((G: Graph)::Node : (Trait: Kind))::method() // c?
```

- niko + acrichto: We think the (A:B:C) case above is resolvable (worst case scenario, require explicit parens, or type defs to force left-hand side to be an identifier)





# 2014/03/07 struct inheritance

## Requirements

 - tree of types (single inheritance)
 - downcasting
 - thin pointers
 - cheap field access
 - easy upcasting
 
## Design

```
// Virtual structs are unsized but *can't* end with an unsized field (so they can be extended)
virtual struct Base { }
impl Base {
    virtual fn foo(...);
    virtual fn bar(...) { }
}

virtual struct Foo: Base { }
impl Foo {
    // Q: How to declare overrides? 'virtual override'?
    virtual override fn foo() { }
}
// Leaves don't need to be virtual (and get static dispatch for virtuals)
struct Baz: Foo { }

```

* base virtual structs don't need to implement virtual methods

## Questions

* How do substructs  instantiate out-of-module super structs with private fields? Can't. For private fields you need to have a closed hierarchy in a single mod.
* 'virtual' required for override?


# 2014/03/06 RFC process

* folders: 'active', 'complete', 'dead'.
* what requires an RFC?
  - major changes need RFC
* fill in RFC template
* open a PR against rust-lang/rfcs
   - 0000-built-in-kinds.md
   - rename file name to match PR # and push -f
* discussion and design on PR
* possible discussion at meeting prior
* core team must merge or reject RFC
* merging PR means...
  - green light to start prototyping / implement
  - does *not* mean a patch implementing the feature will be accepted
* revisions no accepted RFC's should be done as followup PR's editing same RFCs
* obsolete RFCs can be renamed to `rip-`




