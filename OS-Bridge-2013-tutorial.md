These are Tim's notes for a proposed Rust tutorial at [Open Source Bridge](http://opensourcebridge.org/) 2013.

Length: 1 hour, 45 minutes. 15 minutes for Q & A; plan for 90 minutes. ~ 45 slides, two minutes per slide.

* What is Rust? (4 slides)
    * Why another programming language?
    * For systems programming, want a straightforward mapping between language semantics and machine, like what C gives you.
    * For writing safe and reliable applications, don't want buffer overflows, null pointers, or other preventable vulnerabilities.
    * For taking advantage of modern hardware, want usable concurrency constructs (not the focus of this talk)
    * In general, want types and pattern matching, without the overhead of functional languages like Haskell and ML (but we pay the price of having to think more about memory allocation and data representations than in those languages)

