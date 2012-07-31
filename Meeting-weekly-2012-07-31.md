# Agenda

* internship status / progress, documentation (graydon)
* macro_rules! live, start switching over (graydon / pauls)
* crate hashes all changed (graydon)
* parentheses for macro invocations
* structural records
* explicit self
* report on modes
* static methods
* the 'static' region/lifetime (vs. 'const'?)  (graydon)
* tweaks to the lint pass
* ... other items? 
* possibly post-meeting, FTEs stay around to talk syntax (graydon)

# Internships

- People seem to have 4, 4, 6, 4, and 2.8 weeks left
- Before the deadline looms, all interns should take a breather from coding to write down what they've done
- Write in sufficient detail that somebody who comes along later would be able to fix bugs!

# Macros

- pauls: Usable now, going to work on hygiene
- pauls: please use foo! syntax not #foo
- brson: Can they be imported?
- pauls: No, but there is a vague plan, probably a separate resolve-like pass
- brson: Can you write down how you expect that to work?
- pauls: I'll write down what I have, though resolving in general is mysterious
- graydon: that seems to interact with #! "pre-flight-control" and other features of the compiler
- pauls: I'm planning on adding macros to the tutorial in some form today

# Crate hashing

- graydon: We changed from SHA1 to siphash, this required make clean
- brson: What does this buy us?
- graydon: Trying to cull down our set of hash functions
- graydon: weak hash functions are a security vulnerability
- graydon: SIPHash is very modern, useful, and so we are centralizing on that one
- brson: Will we use this for our hashtables?
- graydon: Yes.
- graydon: Main concern is not speed but just consolidating on hash functions
- graydon: SHA1 is nearing the end of its functional life
- ... some discussion on how we can make SIPHash more magic for use in hashtables ...

# Parentheses for macro invocations

- pcwalton: we can't write #debug() anymore
- pauls: we should be able to select delimiters in a relatively easy fashion
- pauls: we chose to restrict it because we were wondering if none-curly-brace arguments might be restricted to expressions
- pcwalton: so basically just accept any delimiters?
- pauls: if were to give multiple delimiters different meanings, the main goal would be to be able to create macros that may take multiple arguments that are not enclosed within a single set of delimiters
- graydon: I am not profoundly concerned with whether we can produce the entire syntax of the language via macros (dryly)

# Structural records

- pcwalton: I would like to remove them from the language
- pcwalton: I've talked with most people about this at some point or another
- pcwalton: Most compelling reason is that they are not especially compatible with typeclasses
- pcwalton: Not a widely documented interaction
- pcwalton: incompatible with forbidding "orphan" implementations, because every time you create a trait on a structural record it's an orphan unless you also defined the trait
- pcwalton: (orphan == impl not in the module of the type nor the trait)
- pcwalton: end goal that we want is that users can define structs with methods and then not have to import anything to use them, which works fine with structs but is not attainable with structural records
- graydon: I am not opposed to removing structural records, however we do have the same problem with vectors and other types.  If users want to define new methods on vectors (or tuples, or other structural types) they will need to define a new trait. So the problem is not *entirely* solved.
- graydon: What do we do with regard to specialization? Like if you define a method on [T]?
- pcwalton: Right now you get an error.  The precise errors depends on how these overlapping instances come about:
    - if you defined the trait, you would have a coherence error, because you have overlapping instances implementing the same trait
    - if you did not define the trait, then you can't implement it anyway, it must be an orphan
- pcwalton: other advantages to structural records, leave off fields, leave off mutable,;accommodate privacy, recursion
- pcwalton: Any objections?
- sully: How do you write the literal form?
- pcwalton: `RecordName { field1: ..., field2: ... }`
- sully: Uglier.
- pcwalton: Maybe.  Another option might be inferring the record name based on various criteria.
- pcwalton: In pattern matching we can use the expected type.
- brson: Does this have any affect on the types that can be recursive?
- pcwalton: Yes, records can be recursive without any enums.

# Explicit self

- pcwalton: Some discussion about this but I wanted to bring it up more widely and now we have something more concrete
- pcwalton: Some idea what the general constraints are on the design
- pcwalton: Proposed syntax: 

    ```
    impl Foo { // Foo is the type
        fn f(&self, x: int) { ... }
    }
    ```

- Defines a method `f()` where the self parameter would be of type `&Foo`
- Other possibilities:

    ```
    fn f(&mut self, x: int) {...} // Receiver of type &mut Foo
    fn f(self, x: int) {...} // Receiver of type Foo
    ```

- Main difference is that today you define `impl &Foo` and then methods for that, but now you would be restricted to `impl Foo` where `Foo` is a nominal type
- Pythonic inspiration
- brson: Does this interact with vector types?
- pcwalton: Unclear, &self becomes a slice perhaps?
- pauls: Would you be required to have a sigil?
- pcwalton: No, particularly useful in the case where the method moves self
- pcwalton: Also helpful for "dual-mode" data structures like nmatsakis has blogged about
- graydon: Those are potentially huge
- pcwalton: Other possible syntaxes, C++ and Go
- graydon: I like the Pythonic one and the reason I like it is that it might be nice to allow `x.f(y)` to a kind of be sugar for `f(x, y)`
- pcwalton: Yes, sully has brought this up.  The big difference is namespacing: x.f(y) allows for type-based overloading of `f` (summarized)
- graydon: Sometimes people might prefer to write in the function-like style so maybe we can allow `f` to be imported?
- pcwalton: We could in principle allow methods to "leak out" into the surrounding scope as functions, but then we'd potentially have multiple names in the module
- graydon: It's not a big deal, but it would help to address these functions like `vec::len()` vs `vec.len()`
- pcwalton: it does kind of dovetail with the idea of having functions in impls that define functions that do not actually take an instance of the receiver
- pcwalton: useful in some cases, like matrix math parameterized over a `Num` typeclass, where you want an `fn identity() -> Num`
- sully: also for building up vectors and so forth
- nmatsakis: basically generic construction
- sully: more compelling than Monoid
- pcwalton: it's already come up with `to_mut()` and `from_mut()` in vectors

# Report on modes

- nmatsakis: will send e-mail

# Static methods

- sully: I want to add them.  
- pcwalton: with explicit self, static fns just fall out, not backwards compatible
- pcwalton: temporarily they could be designated with `static` to make them backwards compatible
- pcwalton: but maybe it's better to just go through and convert and do a snapshot
- sully: I would prefer to avoid having to edit every file in the compiler... again

# The 'static' region/lifetime (vs. 'const'?)

- graydon: I am working on constant evaluation this week
- graydon: It would be nice to have constant vectors and so forth
- graydon: Right now these all have `static` lifetime
- graydon: That's a word that doesn't get much use (`static`)
- graydon: I'm wondering if it confuses things to have a `const` lifetime
- graydon: It's not really a keyword, right, just a reserved lifetime name?
- nmatsaks: Yes

# Tweaks to the lint pass

- graydon: I adjusted the lint pass to have a new syntax
- graydon:
        - `warn(name)      // issue warning`
        - `allow(name)     // no warning`
        - `deny(name)      // issue error`
        - `forbid(name)    // issue error and do not allow it to be re-enabled`
- pcwalton: nmatsakis and I have talked about a very lightweight effect system based on Scala's lightweight effects to allow you to specify no-gc and similar things on a task basis, rather than module level (ed: also works on a function by function level)
- graydon: statically turning off gc on a module-by-module basis is an easy first step, but that's an interesting thing to keep on the back burner

# Syntax discussion (FTE-only just to keep us from bikeshedding too much)

- Keywords under discussion:
    - `ret` to `return`
    - `mod` to `module`
    - `alt` to `match`
    - `pub`, `priv` for access control
- graydon: I'd prefer not to have contextual keywords but just keywords
- Vetos for replacing `ret` with `return`? None
- Vetos for replacing `mod` with `module`?
    - pcwalton: I am a little concerned that you have to write `extern module std;`
    - nmatsakis: that doesn't seem noticably worse than `extern mod std;`
    - Vetos? None
        - graydon is slightly more negative but not enough to veto
        - nmatsakis / brson: don't care
    - pcwalton: I want to use `mod` as a function name
    - graydon: there is an operator for that: `%`
    - pcwalton: I like how `module` looks
    - Trade `module` for `pub` and `priv`
- Access control: `pub` and `priv`
  - is 'pub fn main(args: [str])' too chatty?
- `case` vs `match`
        - pcwalton: prefers match
        - nmatsakis: indifferent
        - brson: prefers match
        - graydon: prefers case (shorter, more commonly used)
- `::` will stay
- CamelCase for types, variants, and traits (Python rules)
- Fat arrow for `match`?
    - general agreement to make it mandatory
    - optional braces
- Semicolons
    - pcwalton: ignore trailing semicolons
    - graydon: the current rule feels very natural to me but a lot of new users don't like it
    - graydon: I have also written a lot of Ocaml and been annoyed by ignore and trailing unit
    - pcwalton: I like the fact that it catches bugs
    - brson: these are separable, right?
    - nmatsakis: to some extent
    - graydon: can we do this by treating blocks that do not use their return value specially
    - experimentation required
- Lexical level:
    - keywords set in stone as of 0.4