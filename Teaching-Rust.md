Firstly, we've lately been thinking about how to introduce Rust to 
experienced programmers and these are the topics, in roughly this order, 
that we believe are critical to teach:

- Stack vs. heap, values, copying
- Ownership (front and center)
- Borrowing
- Copying vs. Moving (and the notion of Pods for copying)
- References vs. values, and their relationship to ownership and 
stack/heap allocation
- Lifetimes (specifically taught in terms of returning borrowed fields 
of a struct)

It's impossible to understand Rust without grasping these concepts, and 
the earlier and more frequently they are taught the better.

Then there are a number of practical subjects that all Rust programmers 
need to know to write real code:

- Vectors vs. Slices
- Structs, enums, and pattern matching
- Also covered in passing, with links to guides for further info: fns, 
impls, basic types, using libraries, `Option`, generics

For an example of how to teach Rust I recommend watching [Niko's recent 
talk](https://t.co/aaYgMqZprC)

# Rust relating to other languages

One good resource is Rust for [C++ programmers].

I consider Rust to be a multi-paradigm systems language, not 
specifically functional, not specifically OO, but drawing influence from 
both. The biggest influences are probably C++ and ML. It's design is 
influenced by existing languages, but more importantly by practical 
experience solving problems of memory safety, performance, and 
concurrency in large-scale software (Servo). To that end it deviates 
from precedent established in other mainstream programming languages, 
introducing the key concepts of ownership and borrowing.

[C++ programmers]: https://github.com/mozilla/rust/wiki/Rust-for-CXX-programmers