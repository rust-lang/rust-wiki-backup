This page is intended to describe the design rationale for Rusts numeric traits, right now it is at bikeshed level but progress is tracked in bug [#4819](https://github.com/mozilla/rust/issues/4819) and when there is more consensus, we'll move it into the documentation/implement it.

### Issues ###

 1. We cannot provide a default implementations for methods due to bugs in rustc I think, something to do with structs and trait inheritance &c.
 2. We cannot inherit generic traits due to bugs in rustc.
 3. We cannot split this out to a `libmath` because of rusts treatment of `#[lang]` items.

## Base Traits ##

These traits are useful for all numeric types, and some are useful to all types (Eq). They form the base of the trait structure, and influence all others.

### cmp::Eq ###

`Eq` is a language item that Rust code may implement to overload the `==` and `!=` operators, and should be implemented by all numeric types.

~~~
trait Eq {
  fn eq(&self, other: &Self) -> bool { return !self.ne(other) };
  fn ne(&self, other: &Self) -> bool { return !self.eq(other) };
}
~~~


### cmp::Ord (Ordered) ###

`Ord` like `Eq` is a language item that the compiler uses to implement comparison operators. Rust code may implement Ord to overload the `<`, `<=`, `>`, and `>=` operators, and should be implemented by all numeric types that are at least partially ordered.

~~~
trait Ord: Eq {
  fn lt(&self, other: &Self) -> bool;
  fn le(&self, other: &Self) -> bool;
  fn ge(&self, other: &Self) -> bool;
  fn gt(&self, other: &Self) -> bool;

  fn min(&self, other: &Self) -> Self { if (self.lt(other)) { self } else { other } };
  fn max(&self, other: &Self) -> Self { if (self.gt(other)) { self } else { other } };
  fn clamp(&self, min: &Self, max: &Self) -> Self { self.min(max).max(min)) };
}
~~~

`min`, `max` and `clamp` could be factored out of this trait if we want to use it for non-numeric types where they do not make sense, but since they can have default implementations and what they do is very intuitive the extra API pressure for those (few) types would be minimal.

#### Min ####

Calculates the minimum of the object and `other`.

#### Max ####

Calculates the maximum of the object and `other`.

#### Clamp ####

Ensures that the object is between the limits set by `min` and `max`. If `min` is greater than `max` the result is undefined.


### Abs<Result> ###

Abs should always take a type parameter Result that is a type that represents a real number but does not necessarily have to implement the Real trait (but really should implement at least `Ord`).

 1. For non-complex numbers, Result should be the type Self.
 2. For complex numbers, Result should be the type used internally to represent the real and imaginary parts. For example, a `struct Complex { real: float, imag: float }` type would use the Result parameter `float`.
 
~~~
trait Abs<Result> {
  fn abs(&self) -> Result; // NOTE: Overflows as normal operations, abs(INT_MIN) = INT_MIN.

  fn abs_min(&self, other: &Self) -> Self;
  fn abs_max(&self, other: &Self) -> Self;
}
~~~

`abs_min` and `abs_max` are from IEEE754-2008 and can be useful when sorting, an example could be that we calculate eigenvalues, and since we want to show only the most influential ones, we sort them by magnitude.

There is some risk for overflow, and it isn't obvious how it should be handled, but I propose we follow current semantics and overflow instead of saturate. I would love to add another trait at a later date with saturating arithmetic, which is awesomely useful, but I think most people could be even more surprised by some operations saturating and some overflowing.

From a performance perspective, the result could just as well be unsigned, like the corresponding hardware instructions in SSSE3 for example. Since that is bitwise equivalent to overflowing, it makes no difference.


### Signed ###

Signed should be implemented by all signed (real) types.

~~~
trait Signed: Neg<Self> {
  fn sign(&self) -> Self;
}
~~~

Since we have copysign, this should probably return `{ -1, 0, +1}` even for floats, but arguments could allow for it to have copysign semanticts, since floating-point technically doesn't have zeroes in the sense that integer types do.

### Additive and Multiplicative ###

These traits signal something more than just the sum of their subtraits, they imply that the basic numeric operators are implemented in a sane way and follow basic mathematical laws to approximately the same degree as the built-in types.

~~~
trait Additive<RHS,Result>: Add<RHS,Result> Sub<RHS,Result> { }
trait Multiplicative<RHS,Result>: Mul<RHS,Result> Div<RHS,Result> One { }
~~~

### Quasigroups &c. ###

Here a quasigroup is a structure resembling a group where the 'division' operation always makes sense. It is a good enough approximation of what properties the trait represents.

It means that it is `Additive` or `Multiplicative` with itself, and has an identity element.

~~~
trait AdditiveQuasigroup: Additive<Self,Self> Zero { }
trait MultiplicativeQuasigroup: Multiplicative<Self,Self> One { }
~~~

A better name could be searched for, since it doesn't mean exactly the same thing as it does in algebra but neither does ordered.

Also, neg / recip could be added here, but I am slightly worried about when someone actually would want to use them and not unary `-` / `T::One / x`. The NativeMath module (if added) should contain optimized low-accuracy versions of these operations that can be useful for graphics and so on, but I would be slightly worried if one needed to implement them to add a new type.

### Sqrt ###

The fifth basic floating-point operation, square root. This should be correctly rounded for types where that makes sense, and for types where it doesn't, it should provide the answer that makes sense in the domain it tries to solve.

~~~
trait Sqrt {
  fn sqrt(&self) -> Self;
}
~~~

The `rsqrt` operation is of great importance for computer graphics applications and should therefore also be considered for inclusion in this trait. But on the other hand, the proposed `NativeMath` trait provides exactly that function with higher performance code, at a cost in accuracy. This trade-off might be acceptable for all users of `rsqrt`.


### FusedMultiplyAdd (FMA, FMAC &c.) ###

While this is trivially implementable for integers since Rust doesn't signal on overflow, this trait is meant to be implemented only by types where it is important (f32 and f64). On systems without support for IEEE 754-2008 in hardware (most x86 systems), this should be implemented in software but that can unfortunatly be slightly slow.

Going forward all major platforms Rust runs on (ARM, MIPS, x86 and x86-64) is going to support this operation, and most other platforms either already supports it in hardware, or is adding it already due to its almost magical powers for implementing fast floating-point division, square-root and its amazing powers to increase accuracy in almost every common numerical task.

~~~
trait FusedMultiplyAdd {
  fn fma(&self, a: Self, b: Self) -> Self;
  fn fms(&self, a: Self, b: Self) -> Self;
}
~~~

Could maybe be factored into FusedMultiplyAdd<RHS1,RHS2,Result>?

### Num (Number) ###

This is the trait we've all been waiting for, unfortunately it isn't really that amazing. It is just the sum of a few other traits, but that also means that it is quite generic.

The major properties of a number in Rust is that they support all arithmetic operators, they have identity operators relating to them and that you can compare instances.

~~~
trait Num: Eq AdditiveQuasigroup MultiplicativeQuasigroup { }
~~~

### Real ###

A `Real` is just a `Num`, but in addition is ordered, has a square root and an absolute value that returns the same type, which are quite common requirements.

~~~
trait Real: Num Ord Sqrt Abs<Self> { }
~~~

All integer, floating-point and fixed-point types can trivially implement this. `Sqrt` isn't currently implemented by integer types, but it is useful for some algorithms, so I wouldn't mind adding one.

## Bit- and byte-level Traits ##

Most bit-level traits can be implemented on all types, but they are most commonly associated with integers. 

### Bitwise ###

The main trait is `Bitwise`, which implements all the normal bitwise operations, always returning the same type.

~~~
trait Bitwise: Not<Self>
               BitAnd<Self,Self>
               BitOr<Self,Self>
               BitXor<Self,Self> {

}
~~~

### Shift ###

The second trait is `Shift` which implements shifts in a way that makes 'sense' for the type, arithmetic shifts for signed types, and logical shifts for unsigned types. For most non-integral types, like floating-point for example, logical shifts would be the kind that makes sense.

~~~
trait Shift<RHS>: Shl<RHS,Self> Shr<RHS,Self> { }
~~~

### Bitcount ###

I find these useful, so I added them here. We also currently have unexposed intrinsics for them, which this would fix. Note that `clz` / `ctz` never returns undefined results (I'm looking at you, x86) and if there isn't a binary one in the value, return the size of the type.

~~~
trait Bitcount<I:Int> {
  fn popcount(&self) -> I;
  fn clz(&self) -> I;
  fn ctz(&self) -> I;
}
~~~

### Byte-Swap ###

This trait is useful for when communicating over a network when on a little-endian host (which rust always is at the moment) and also have unexposed intrinsics like the bitcount ones.

The two methods `to_big_endian` and `to_little_endian` should of course have default implementations that you never should override.

~~~
trait ByteSwap {
  fn byte_swap(&self) -> Self;

  fn to_big_endian(&self) -> Self;
  fn to_little_endian(&self) -> Self;
}
~~~


## Method wrappers ##

Using specific method implementations for numeric functions is extremely useful, as it enables us to tailor the methods to the specific type (for example using LLVM intrinsics or libm functions) whilst also being able to provide generic default methods to reduce code duplication (for example with `Ord::abs` and `Ord::clamp`). The downside is the for some mathematical functions the dot syntax is less readable than the functional equivalent. For example `abs(x)` or `min(x, y)` is more readable than `x.abs()` or `x.min(y)`.

The solution is to create a series of inlined generic method wrappers to allow the user to choose which form to use depending on the situation:

~~~
mod math {
    #[inline(always)] fn abs<T:Abs>(x: T) -> T { x.abs() }
    #[inline(always)] fn min<T:Abs>(x: T, y: T) -> T { x.min(y) }
    #[inline(always)] fn sqrt<T:Sqrt>(x: T) -> T { x.sqrt() }
    ...
}
~~~

Sometimes the user might want to be more specific with type information, so it would be useful to include specific wrappers for each of rust's primitive numeric types:

~~~
mod float {
    #[inline(always)] fn abs(x: float) -> float { x.abs() }
    #[inline(always)] fn min(x: float, y: float) -> float { x.min(y) }
    #[inline(always)] fn sqrt(x: float) -> float { x.sqrt() }
    ...
}

mod f64 {
    #[inline(always)] fn abs(x: f64) -> f64 { x.abs() }
    #[inline(always)] fn min(x: f64, y: f64) -> f64 { x.min(y) }
    #[inline(always)] fn sqrt(x: f64) -> f64 { x.sqrt() }
    ...
}
~~~

The type-specific intent would be obvious when calling these functions, for example `float::abs(-0.5)` or `f64::min(2.0, 7.5)`.

## Constants ##

Initially constants could be declared using static methods:

~~~rust
pub trait Float {
    fn NaN()           -> Self;
    fn infinity()      -> Self;
    fn neg_infinity()  -> Self;
    fn pi()            -> Self;    // π
    fn two_pi()        -> Self;    // 2 × π
    ...
}

impl Float for f32 {
    #[inline(always)] fn NaN()          -> f32 { 0_f32 / 0_f32 }
    #[inline(always)] fn infinity()     -> f32 { 1_f32 / 0_f32 }
    #[inline(always)] fn neg_infinity() -> f32 {-1_f32 / 0_f32 }
    #[inline(always)] fn two_pi()       -> f32 { 6.28318530717958647692528676655900576 }
    #[inline(always)] fn pi()           -> f32 { 3.14159265358979323846264338327950288 }
    ...
}
~~~

This is not ideal however, as these 'constants' would not be able to be used in compile time expressions (for example when defining new constants). There has been talk of adding '[Associated Items](http://smallcultfollowing.com/babysteps/blog/2013/04/03/associated-items-continued/)' to traits in the future. This would allow the definition of true constants:

~~~rust
impl Float for f32 {pub trait Float {
    static NAN           : Self;
    static INFINITY      : Self;
    static NEG_INFINITY  : Self;
    static PI            : Self;    // π
    static TWO_PI        : Self;    // 2 × π
    ...
}

impl Float for f32 {
    static NAN          : f32 =  0_f32 / 0_f32;
    static INFINITY     : f32 =  1_f32 / 0_f32;
    static NEG_INFINITY : f32 = -1_f32 / 0_f32;
    static TWO_PI       : f32 =  6.28318530717958647692528676655900576;
    static PI           : f32 =  3.14159265358979323846264338327950288;
    ...
}
~~~

## Performance ##

### LLVM Intrinsics ###

LLVM provides [various intrinsic functions](http://llvm.org/docs/LangRef.html#intrinsic-functions) that could be used in the numeric trait implementations. These are already in the rust compiler, so using them should be quite easy, and reduce our dependence on `libm` / `libc`, possibly increase performance and improve cross-platform numeric compatability.

### Native Math ###

The functions here are copied from OpenCL, and are intended to map to lower level instructions, and are (possibly) not as accurate as the normal ones. The idea is that many applications (graphics mostly) does not need highly accurate functions, but can work fine with much faster, lower precision versions.

~~~
trait NativeMath {
  fn native_cos(&self) -> Self;
  fn native_divide(&self, &b: Self) -> Self;
  fn native_exp(&self) -> Self;
  fn native_exp2(&self) -> Self;
  fn native_exp10(&self) -> Self;
  fn native_log(&self) -> Self;
  fn native_log2(&self) -> Self;
  fn native_log10(&self) -> Self;
  fn native_powr(&self, &b: Self) -> Self;
  fn native_recip(&self) -> Self;
  fn native_rsqrt(&self) -> Self;
  fn native_sin(&self) -> Self;
  fn native_sqrt(&self) -> Self;
  fn native_tan(&self) -> Self;
}
~~~

## Alternative idea: Add more generics to numerical traits ##
The following is a lose idea which may or may not work.

~~~
trait AllNumOperations <A: One + Zero, B > Multiplicative<A, B> + Additive<A, B>
trait CommutativeOperations <A : AllNumOperations<Self, B>, B> : AllNumOperations <A, B>;
~~~

The benefit of this approach is that you can specify things like 

~~~
impl CommutativeOperations<Int, F32> for F32 { .. };
impl CommutativeOperations<UInt, F16> for F16 { .. };
impl CommutativeOperations<F32, SymbolicExpression> for SymbolicExpression { .. };
impl AllNumOperations<SparseMatrix, FullMatrix> for FullMatrix { .. }; 
~~~

Hopefully this will help the user to do less casting. Similarly it may be useful to compare ints with floats, so perhaps the Ord trait could be generic as well.

~~~
trait Ord < A >: Eq < A >{
  fn lt(&self, other: &A) -> bool;
  ...
}
~~~

Lastly it might be useful to disallow integer division, because this is a common source of error. (A function of two variables would be more explicit) 

~~~
trait NumOpExceptDivison <A, B : One + Zero> Mul<A, B> + Additive<A, B>
impl NumOpExceptDivison <uint, uint> for uint {..}
~~~

## String Representation ##

There is also the question about how to handle the  `ToStr`, `FromStr`, `ToStrRadix`, etc traits:

- `core::num::strconv` contains traits and functions to convert to/from a string in fractional positional notation
  - useful for all scalar numbers
  - should ideally get impl for free on all those numbers though the appropriate numeric traits
- need functions for different formats (`1/2` vs `0.5`, `4+3i`)
- getting a unified interface for automatic formatting (ala the current `NumStrConv`) is probably an impossible task, since there are so many different sorts of numbers. i.e. it's almost certainly not possible to make a single scheme general enough to include ints, floats, rationals, complexs, finite fields/integer modulo rings, algebraic numbers, [exact real arithmetic](http://www.haskell.org/haskellwiki/Exact_real_arithmetic), etc, so it would be good to decide on a limit and let other numbers implement their own conversions, rather than try to bolt on extensions/squeeze numbers into a framework they don't fit
- possible new traits:
  - `ToStrExact` which prints in a format that can be converted back without any precision loss (like `fmt!("%?", ..)`, but nicer, e.g. 1/10 for rationals)

## Call LLVM Directly From Operator Overloads ##

The operators for primitive types are currently hard-coded into the compiler. This leads to the operator overloads looking rather ridiculous:

~~~rust
impl Add<T,T> for T {
    #[inline(always)] fn add(&self, other: &T) -> T { *self + *other }
}
~~~

It would be more elegant if we could call LLVM directly from the operator trait:

~~~rust
impl Add<T,T> for T {
    #[inline(always)]
    fn add(&self, other: &T) -> T {
        unsafe { llvm!( "%0 = add u8 %1, u8 %2"
                      : "r="(result)
                      : "r"(a), "r"(b)); result") }
    }
}
~~~

This would allow us to remove the operator code from the compiler.

## Related Changes ##

### rename `modulo` into `rem` or `remainder` in traits and docs ###

Rusts `%` operator mimics C and C++ in that it's not modulo but remainder, however the documentation and naming convention claims for it to be modulo.

## See Also ##

[Diagram of Haskell's numeric type heirachy](http://www.bucephalus.org/text/Haskell98numbers/Haskell98numbers.png)

[Discussion of this page on reddit]
(http://www.reddit.com/r/rust/comments/18btxy/bikeshed_numeric_traits/)