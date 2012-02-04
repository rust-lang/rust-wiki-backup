This is a proposal for a lightweight bind syntax which (basically) omits the keyword bind.  I (nmatsakis) have implemented a prototype which I will describe here.  I am unsure about some of the particulars, however, and wanted to receive feedback. -- Niko Matsakis

## Goals

I want to make it possible to write things like:

    points.map(_.x) // extract a list of all the x coordinates

I also want to bind to work for nested closures.  I intend to use this in an iteration library and possibly in the task library.  However, I am not sure whether this iter library is the best design,and this particular case leads to a kind of inconsistency in the syntax, see discussion below:

    pointers.iter(_).map(_.x, _).to_vec()

I want to ease mode mismatch.

    fn add(x: int, y: int) -> int { x + y }
    [1,2,3].map(add(3, _)) // currently yields an error, works in my impl

Finally, on a non user-facing note, I want to simplify the implementation of bind so it is easier to support going forward.  For example, the existing bind does not support method calls, has some typestate bugs (which I discovered while implementing), and in general takes an almost completely different path from other kinds of closures.  In my implementation, both binds and `{||...}` closures are represented almost identically and can make use of the same code.

## Status

The implementation permits bind expressions with the following forms:

```
_.a.b.c        => {|x| x.a.b.c }
_(_, _)        => {|x,y,z| x(y, z) }
a.b(_, _)      => {|x,y| a.b(x, y) }
_.a.b.c(_, _)  => {|x,y,z| x.a.b.c(y, z) }
```

A naked `_` is an error: it can only appear as a hole in a larger expression.  Binary operators (`_[_]`, `_ + _`) are not currently allowed but would be easy to support.  We could also allow the conditions of expressions like `if _ { a } else { b }` to be omitted.

These expressions can be nested, so:

```
_(_.a)    => {|x| x(_.a)}    => {|x| x({|y| y.a})}
a(_).b(_) => {|y| a(_).b(y)} => {|y| {|x| a(x)}.b(y)}
```

## Side benefits and side *effects*

The consolidated code path is very nice.  I also improved inference for all closures.  Regardless of what happens with the proposal, I will commit the improved inference.  However, the consolidated code path has one subtle side effect.  Under the old bind, supplied function arguments were evaluated when the closure was created.  Under the new system, supplied function arguments are evaluated each time the closure is called.  

To see the difference, compare the expansion of the old-style bind and the new-style:

```
bind f(a.b, _) => { let x = a.b; {|y| f(x,y) } }
f(a.b, _) => {|y| f(a.b,y) }
```

Old-style bind is closer to classic currying.  New-style is basically a shorthand.  I could fix this, but it would largely erase the gains of a consolidated code path.  I don't yet see an obvious way to keep a consolidated code path *and* the old semantics.  

Moreover, for things like `a.b(_)`, where `b` is a method, it is precisely this change which makes this work whereas before it failed.  This is because there is no need to reify a "about to be invoked" method.  Basically this work "fixes" [issue #435][435] by circumventing the problem.  Syntax like `a.b` where `b` is a method could then just be made illegal.

[435]: https://github.com/mozilla/rust/issues/435

## Questions and concerns

#### 1. Should other expression forms be supported?  Which ones?

In particular I think binary operators can be useful,
especially if we allow for method overloading.  Something like
`scores.foldl(0, _+_)` (which would sum all of the scores)
reads fairly well to me.  What do we want to allow?

#### 2. Do we care about the change in evaluation order vs `bind`?

I don't, particularly, but the old order corresponded more to classic
currying.

#### 3. Nesting is somewhat inconsistent.

In general, I tried to say that a plain `_` indicates a hole in the
expression in which it appears, but a nested `_` expression creates
a nested closure.  So `_(_)` yields `{|x,y| x(y)}` but `_(_.a)`
yields `{|x| x(_.a)}` (as shown above).  However, this nesting rule is
not 100% consistent: the receiver of a call is not considered
nested, but calls are.  So:

```
_.a(_) => {|x,y| x.a(y)}
_.a(_).b(_) => {|z| _.a(_).b(z)}
```

Unfortunately, because we do not know syntactically whether the `a.b` in
`a.b()` is a method call or a field access. I could either 

1. leave it this way;
2. accept `_.b(_)` but only if `b()` turns out to be a method; or
3. translate nested calls like this:
   ```
   _.a(_).b(_) => {|x,y,z| x.a(y).b(z)}
   ```
   This option however does not mesh with the iteration library I had in mind.

#### 4. Is it too magical?

I feel like I'm always tweaking the syntax to make
it more complex.  Odd because I think of myself as preferring simple, regular syntax, as
long as it's not S-expressions (sorry Dave).  But perhaps I don't know myself that well!
Anyhow, I do think `{|x| f(x)}` and `{|x| x.f}` are significantly less readable
than `f(_)` and `_.f`.