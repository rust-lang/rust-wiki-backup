These are Tim's notes for a proposed Rust tutorial at [Open Source Bridge](http://opensourcebridge.org/) 2013.

Length: 1 hour, 45 minutes. 15 minutes for Q & A; plan for 90 minutes. ~ 45 slides, two minutes per slide.

Examples: Look at Servo (resource manager, display list...), https://github.com/pcwalton/fempeg , https://github.com/pcwalton/sprocketnes , ...

* What is Rust? (4 slides)
    * Why another programming language?
    * For systems programming, want a straightforward mapping between language semantics and machine, like what C gives you.
    * For writing safe and reliable applications, don't want buffer overflows, null pointers, or other preventable vulnerabilities.
    * For taking advantage of modern hardware, want usable concurrency constructs (not the focus of this talk)
    * In general, want types and pattern matching, without the overhead of functional languages like Haskell and ML (but we pay the price of having to think more about memory allocation and data representations than in those languages)
* Fun with types (4 slides)
    * Algebraic data types (simple example)
    * Generics (ditto)
    * Pattern matching
    * Structs: named product types
    * Field access, functional record update
* Using traits (4 slides)
    * intro to traits (simple example)
    * trait inheritance
    * default methods
    * objects / dynamic dispatch
    * the difference between static and dynamic dispatch
Could start with a program that duplicates code, and show how traits emerge when refactoring it
* Background on traits (2 slides)
    * Traits vs. type classes
    * Traits vs. OO classes
    * bringing together OO & functional styles
* Pointers and memory in Rust (10 slides)
(example for each)
    * In Java, Haskell, ... , everything (except for a few primitive types) is implicitly boxed, and you can't explicitly create/dereference pointers
    * Not so in Rust: in fact, we have three kinds of pointers
    * Shared (@)
    * Unique (~)
    * @ is garbage-collected, ~ gets freed automatically because the compiler knows its lifetime
    * You can send ~ in between tasks (haven't introduced tasks yet)
    * Borrowed (&)
    * Interior pointers
    * Returning a borrowed pointer into a structure
    * The important thing is that this is all safe: no dangling pointers, no leaks. (except with unsafe code or compiler/RT bugs, handwave handwave)
(24 slides so far)
* A more extended example combining traits and pointers? (4 slides)
* A more extended example exhibiting borrowed pointers? (4 slides)
* Miscellaneous fun stuff (4 slides):
    * #[test]
    * #[bench]
    * fmt! (nod to macros in general); deriving-eq?
    * (what else?)
* Concurrency (3 slides) (maybe... very simple in any case)
    * example (maybe from one of the shootout benchmarks)
39 so far
* Demonstrate changing a simple program with @-boxes into something with no GC? Show stats for amount of memory allocated?
* Questions?