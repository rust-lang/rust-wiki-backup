## Constructor expressions

The aim of this proposal is to make the DPS optimization more
transparent to end users and simultaneously eliminate implicit copies
of (potentially) mutable state.  The gist of it is that when aggregate
types (most notably records) are used as values they cannot be
assigned to by any arbitrary expression but rather by a limited subset
of expressions known as *constructor expression*.  A constructor
expression is one which constructs a new value.  The compiler will
compile an assignment to an aggregate value type from a constructor
expression by first allocating the space for the aggregate value type
and passing a pointer to that space to the constructor expression, so
that the resulting value is written directly into the appropriate
space with no copies.  This is the same as DPS-optimized code today.

A secondary goal is to permit Rust functions to (usually) follow the
more traditional ABI where return values are returned in a register
rather than adding an out param.  It is believed that this will be
more efficient in the long run, but this assertion has been challenged
and deserves measurement.

## Distinguish constructor expressions

Whether an expression is a constructor expression or not is not purely
a grammatical question, it requires some semantic judgement.   Here is
the set of potential constructor expressions:

    CE = { ... }                 (Record literals)
       | [ ... ]                 (Array literals)
       | 0, 0u, 'a', 1.2         (Numeric/character literals)
       | "..."                   (Strings)
       | ( ... )                 (Tuples)
       | x(...)                  (Function call with new return type)
       | copy(E)                 (Copy of some other expression E)
       | move x                  (Move of some local variable x)

Constructor expressions fall into three primary categories: literals,
(certain) function calls, and copies/moves, each discussed in its own
section below.

### Literals

Literal expressions are fairly straightforward: they are generating a
new value and this value can be written into the appropriate
destination.

### Function calls with new return type

Not all functions can serve as a constructor expression.  The constructor's
return type must make use of the `new` keyword:

    fn foo(...) -> new T { 
        ret CE;
    }

Semantically, the `new` keyword signals that the function returns a
freshly constructed or copied value.  The caller of the function is
able to specify where that value will be stored.  Functions with `new`
return type must themselves return a constructor expression.

At a lower level, the `new` keyword signals that the function, when
compiled, uses a different ABI.  The result is not returned in a
register but rather via an implicit parameter.  Generally this
parameter is pointer to the destination but it could also be an
allocator function; more details on the ABI question are in the
[[Proposal for fixed length arrays]].  

### Copies and moves

As a sort of escape hatch, the `copy` and `move` keywords can be used
to convert any expression into a constructor expression.  The keyword
`copy(expr)` evaluates the expression `expr` and then performs a deep
copy according to the following rules:

- Copying a scalar like `int` just returns the scalar.
- Copying an aggregate value type `T` recursively copies the contents of T.
- Copying an `@T` type increments the ref count and returns the same
  pointer.
- Copying a unique type `~T` creates a box on the exchange heap and
  copies the contents of `T` into that box.
- Copying a reference `&T` simply copies the pointer.
- Copying a resource is illegal.

The `move x` expression is similar in some ways to a copy followed by
a nullification of the local variable `x`.  The primary difference is
for unique types and resources:

- Moving a unique type simply copies the pointer and performs no
  recursive copies.
- Similarly, moving a resource is legal.  

The details are in the [[Proposal for unique types]].

## Interaction with generics

The user can never be sure that generic types are not bound to value
types.  Therefore, uses of variables of generic type generally require
explicit `copy` or `move` annotations; however, the last use
optimization described by Marijn means that a variable which is only
used once is implicitly moved.

> Examples:
>
>     fn identity<T>(t: T) -> T { ret t; } // ok because implicitly ret move t;
> 
>     fn apply<S,T>(s: S, blk: block(S) -> new T) -> new T { 
>         ret blk(s); // implicitly: ret blk(move s); 
>     }
>
>     fn apply_twice<copy S,T>(s: S, blk: block(S) -> new T) -> new (T,T) {
>        ret (blk(copy(s)), blk(s)); // Note: first use of `s` requires a copy
>     }

## Possible changes

The `new` keyword is not strictly needed.  One could say that all
functions which return an aggregate or generic type make use of an
out-param.  In that case, the distinction between `new` functions is
less transparent to the user and is more the concern of the runtime.
The rest of the proposal works precisely the same.

