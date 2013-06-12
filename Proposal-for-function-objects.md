# Proposal for function objects

## Introduction

This proposal replaces the current closure types with function objects, which are simply normal objects which implement a "callable" trait. This idea is based on C++, where any type that implements `operator()` becomes a callable function object.

To make transition to the new types easier, the `fn` keyword is repurposed to define 3 trait types.trait 
```rust
// Can be called once, this is the most general function type
trait once fn(Args...) -> Ret {
	fn call(self, Args...) -> Ret;
}

// Can be called multiple times, but may mutate its state
trait fn(Args...) -> Ret {
	fn call(&mut self, Args...) -> Ret;
}

// Can be called multiple times and has immutable state
trait const fn(Args...) -> Ret {
	fn call(&self, Args...) -> Ret;
}
```
Because these are not valid in the current trait syntax (use of `fn`, variadic type parameters), they will probably have to be defined by the compiler.

The 3 function traits form a hierarchy: A `fn` can be called once and a `const fn` can be called with a mutable object. This results in a supertrait-like relationship between the traits, which also allows one-way implicit conversions: `const fn` -> `fn` -> `once fn`. This allows a function taking another function as a parameter to specify the minimum requirements it needs from the function (Can it be called multiple times? Does it have mutable state?)

## Pointers to

Similar to the auto-dereferencing of the dot operator, pointers to a function object are also callable, which means that the pointers themselves implement the `fn` traits:
```rust
// Mutable reference to a fn are callable
impl<F: fn(Args...) -> Ret> fn(Args...) -> Ret for &mut F {
	fn call(&self, Args...) -> Ret {
		(**self).call(Args...)
	}
}

// Both mutable and immutable references to a const fn are callable
impl<F: fn(Args...) -> Ret> const fn(Args...) -> Ret for &mut F {
	fn call(&self, Args...) -> Ret {
		(**self).call(Args...)
	}
}
impl<F: fn(Args...) -> Ret> const fn(Args...) -> Ret for &F {
	fn call(&self, Args...) -> Ret {
		(**self).call(Args...)
	}
}

// References to a once fn are not callable, it must be called with a value
```

Type bounds on the `fn` keywords will also be supported. They will have the same behavior as defining a new trait which inherits from both the `fn` trait and the listed traits:
```rust
once fn:Owned()
-- is equivanted to --
once fn()+Owned
```
This is currently necessary because there is no syntax to define a trait object which combines 2 or more existing traits (which the + operator provides when defining bounds on generic type paramters).

## Use case examples

Here is what the use cases described in http://smallcultfollowing.com/babysteps/blog/2013/05/13/recurring-closures-and-dynamically-sized-types/ would look like:

1. Higher-order functions
```rust
// Currently
fn each<'r, T>(vec: &'r [T], f: &fn(&'r T))

// Proposed
fn each<'r, T, F: fn(&'r T)>(vec: &'r [T], f: F)
```

2. Once functions
```rust
// Currently
fn each<'r, T>(opt: &'r Optional<T>, f: &once fn(&'r T))

// Proposed
fn each<'r, T, F: once fn(&'r T)>(opt: &'r Optional<T>, f: F)
```

3. Sendable functions
```rust
// Currently
fn spawn(f: ~fn())

// Proposed
fn spawn(f: ~fn:Owned())
```

4. Sendable once functions
```rust
// Currently
fn spawn(f: ~once fn())

// Proposed
fn spawn(f: ~once fn:Owned())
```

5. Const functions
```rust
// Currently
fn each<'r, T>(vec: &'r [T], f: &fn:Const(&'r T))

// Proposed (the const bound must be used to ensure the type does not include &mut T pointers)
fn each<'r, T, F: const fn:Const(&'r T)(vec: &'r [T], f: F)
```

6. Sendable const functions
```rust
// Currently
fn spawn(f: ~fn:Const())

// Proposed
fn spawn(f: ~const fn:Const+Owned())
```

7. Combinators
This is something which is not currently possible with the current type system, but would be possible if arbitrary types can become callable.
```rust
// Structure containing a function and a bound value
struct bind_result<F: once fn(int)> {
	func: F,
	val: int
}

// Make bind_result a callable object, inheriting the same function type as F
impl<F: once fn(int)> once fn() for bind_result<F> {
	fn call(self) { self.func(self.val) }
}
impl<F: fn(int)> fn() for bind_result<F> {
	fn call(&mut self) { self.func(self.val) }
}
impl<F: const fn(int)> const fn() for bind_result<F> {
	fn call(&self) { self.func(self.val) }
}

// A binder which binds an int to the first parameter of a function object, and returns a new function object containing the old one
fn bind<F: once fn(int)>(func: F, val: int) -> bind_result<F> {
	bind_result {func: func, val: val}
}
```

## Sub-proposal: C++11 style closures

This part of the proposal is separate from the above, but does depend on it. I propose that Rust adopts the C++11 lambda syntax to generate closures, which allows fine-grain control of which variables are captured by value and which are captured by reference.

This is best demonstrated by an example:
```rust
fn main() {
	let a: int = 1;
	let b: int = 2;
	let c: int = 3;
	// Creates an anonymous function type which:
	// - captures a by value
	// - captures b by reference
	// - captures c by mutable reference
	// - holds a variable d with the value 4
	// - takes one int parameter
	let func1 = |a, &b, &mut c, d = 4|(e: int) -> int {
		// b and c are automatically dereferenced
		c = a + b + d + e;
		c
	};

	// Some other examples (note that () is optional when there are no parameters)
	let func2 = |=| {a} // = means capture all referenced variables by value
	let func3 = |&, c| {b} // & means capture all referenced variables by reference, except c which is captured by value
	let func4 = || {c} // Error: no default capture mode defined and c is not explicitly captured
}
```

In the previous example, `func1` would generate the following struct:
```rust
// 'r is the intersection of all by-ref lifetimes
struct anonymous_type<'r> {
	a: int,
	b: &'r int,
	c: &'r mut int,
	d: int
}
```

In the majority of cases, a closure will implement the `const fn` trait since its environment is not modified. There are 2 exceptions to this:
- If the closure modifies one of the variables in its environment (note that this does not apply to by-ref captures, since the pointers are not modified), then it will implement the `fn` trait.
- If the closures moves a variable out of its environment, then it will implement the `once fn` trait.

An alternative approach would be to force the environment to be immutable unless mutability or movability is explicitly requested:
```rust
// Mutable closure
|=|(int) mut {}

// Once (movable variables) closure
|=|(int) once {}
```