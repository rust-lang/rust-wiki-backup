# Operator Overloading

We currently (Jan 2012) have a simple form of operator overloading. Interface and impl methods can be named after operators, as in

```
type point = {x: int, y: int};
impl point_ops for point {
    fn +(other: point) -> point {
        {x: self.x + other.x, y: self.y + other.y}
    }
    fn -(other: point) -> point {
        {x: self.x - other.x, y: self.y - other.y}
    }
}
```

When this impl is in scope, and binary + or - are applied to a value of type `point`, the operators will turn into calls to these methods. Operator/type combinations that are defined 'natively' by the language can not be overload (for example + on int).

The operators that can be overloaded are: `+`, `-` (both unary and binary), `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `>>>`, `!` (unary) and `[]` (the index operator). The names of the methods used to implement them correspond to the operator symbols (these operators are parsed as valid method names). For unary minus, the name `unary-` is used.
