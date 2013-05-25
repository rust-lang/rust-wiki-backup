# Operator Overloading

We currently (May 2013) have a simple form of operator overloading. There are a few traits (in `std::ops`), that provide methods that are called for each operator, e.g.

```rust
struct Point {x: int, y: int};

impl Add<Point, Point> for Point {
    fn add(&self, other: &Point) -> Point {
        Point {x: self.x + other.x, y: self.y + other.y}
    }
}

impl Sub<Point, Point> for Point
    fn sub(&self, other: &Point) -> Point {
        Point {x: self.x - other.x, y: self.y - other.y}
    }
}
fn main() {
    Point {x: 1, y: 0} + Point {x: 2, y: 3}
}
```

When this impl is in scope, and binary + or - are applied to two values of type `Point`, the operators will turn into calls to these methods. Operator/type combinations that are defined 'natively' by the language can not be overloaded (for example + on int). The generics on the trait correspond to the right hand side and the result of the operation, which are not required to be the same as the left hand side (the type the trait is being implemented for).

The operators that can be overloaded are: `+`, `-` (both unary and binary), `*`, `/`, `%`, `&`, `|`, `^`, `<<`, `>>`, `!` (unary) and `[]` (the index operator). The names of the methods and traits used to implement them correspond to the names in English (the traits are `Add`, `Sub`, `Neg`, `Mul`, `Div`, `Rem`, `BitAnd`, `BitOr`, `BitXor`, `Shr`, `Shl`, `Not` and `Index` and the methods are just the trait name in lowercase).

The values for an operator are automatically borrowed, so `a + b` is sugar for `(&a).add(&b)`.