## Attending:

Niko, Eric, Patrick, Paul, Lindsey, Dave, Tim, Sully, Ben, Brian, Jesse

## Patrick's vision for upcoming language changes

### Maxmin classes

```
class C { ... }
fn (&C) foo() { ... }
fn (&C) { foo() { ... } bar() { ... } }

```

### Method declarations moved out of line

```
iface show { fn to_str() -> str; }
fn (int) show::to_str() -> str { ... }
```

### Interfaces

  * **Patrick**: coherence (at most one impl per type/iface pair)

### Non-method ifaces

  * **Patrick**: don't have to have a receiver

```
iface read { static fn read(fd) -> self; }
fn read::read(fd 
```

### Replacing kinds with ifaces

```
fn (T) drop::drop(C) { ... }
```

  * **Dave**: tagging ifaces in Java are used sort of like phantom types; if you can't create empty ifaces we can't exactly address that use case

### Operator overloading

```
fn <
fn oper <(...) { ... }
```

### enums/classes

```
enum node {
    mut parent: @node,
    mut first_child: @node,
    .
    .
    .
    element {
        mut tag_name: str;
        .
        .
        .
        a { ... }
        html { ... }
        img { ... }
    }
}
```

  * **Patrick**: declaring sub-enums inline gives the prefix property, avoiding MAX size

### Remove typestate

  * **Patrick**: it's not used

### Case-sensitivity for modules

  * **Patrick**: `::` is uuuugly
  * **Patrick**: capitalize module names, use dot
  * **Niko**: also capitalization for nullary variants? Graydon likely to object to this
  * **Patrick**: people hate `::`
  * **Dave**: we over-qualify even the most common names
  * **Niko**: we have to do that right now, type classes will help improve this
  * **Patrick**: but people still hate `::`
  * **Lindsey**: we also don't have any review yet, so there's no agreed-upon style
  * **Patrick**: there are old passes in rustc that are ugly and unmaintainable because they have to be
  * **Patrick**: we'll need to rewrite all of these eventually
  * **Dave**: needs to be piecemeal
  * **Niko**: trans is a good example that can be done piecemeal
  * **Dave**: syntax redesign is a separate thing from our codebase

### Dynamic-sized types and vectors

  * **Niko**: graydon proposed breaking vectors into 4 kinds:
   * `[]/@` - vector in boxed heap
   * `[]/~` - pointer to unique vector

```
struct vec<T> {
    int fill, capacity;
    T elts[];
}

```

   * `[]/&` - slice

```
struct slice<T> {
    T *elt;
    int len;
}
```

   * `[]/n` - fixed-length

```
T elts[N];
```

  * **Niko**: this is cool, but the notation is confusing; the @ is in the back rather than the front but has the same meaning as front for other cases
  * **Niko**: I'd prefer:

   * `@[]`
   * `~[]`

  * **Niko**: but this is a dynamically-sized type
  * **Niko**: so we bifurcate the type system: some types have known size, some don't; dynamic size types are sort of second-class, have some restrictions
  * **Niko**: my current plan is that you can do almost nothing with them
  * **Brian**: this has something to do with self-describing uniques; we have unique boxes now with a hole in them for type descriptors, and that hole is a thing with an unknown size
  * **Niko**: problem with syntax above, so I proposed instead:
   * `@vec<m T>`
   * `~vec<m T>`
   * `&vec<m T>`
   * `[T]` - slice
   * `(T * n)` - fixed length
  * **Dave**: one problem is it's using common array syntax for a thing that is not quite an array (slice)
  * **Niko**: a few other dynamically sized types: `fn@`, `fn~`, `fn&` can now become `@fn`, `~fn`, `&fn`, and the bounds can be expressed e.g. `~fn:send`
  * **Patrick**: can distinguish `~fn:send` from `~fn:copy send`
  * **Sully**: can you still use block syntax?
  * **Niko**: e.g. `spawn {|| ... }`, yes, that doesn't change at all
  * **Niko**: last case is `ifaces`
  * **Niko**: conceptually simpler, will also clean up a lot of our code

### Closure syntax

  * **Patrick**: old:

```
{|x| x + 1}
uint::range() { |x| ... }
spawn {|| ... }
```

  * **Patrick**: new:

```
|x| x + 1
for uint::range(0, 10) |x| {
    ...
}
do spawn {
    ...
}
```

  * **Sully**: I like that this eliminates the special case of the block-as-implicit-last-argument
  * **Eric**: this is great
  * **Lindsey**: I like this
  * **Dave**: this wins in every way
  * **Patrick**: this actually simplifies the parser
