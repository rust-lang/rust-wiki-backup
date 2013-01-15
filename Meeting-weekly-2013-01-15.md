## Attending

dherman, azita, nmatsakis, jclements, pcwalton, brson, tjc

## Agenda

  * FUEGO!
  * topics on Patrick's whiteboard:
    * move assert and log to macros
    * conditions in `#[cfg()]`
    * `fn() unsafe { ... }`
    * where should we allow trait bounds?
    * unsafe pointer indexing
    * `[ u8 * CONST ]`
    * removal of "both type and trait" impls
    * `Eq` and `Ord` reform

## Tree's on fire

  * **Graydon**: tree's been on fire for over a week
  * **Patrick**: none of us are hitting these LLVM issues and I don't understand them

## assert and log

  * **Patrick**: still built-in; doesn't seem necessary
  * **Graydon**: agreed
  * **Patrick**: question with assert! makes it look like assert-not; had an idea whereby you could rename it failUnless but don't know whether people like that
  * **Patrick**: bang doesn't look like not
  * **Graydon**: wouldn't worry; think you have to understand id! means macro
  * **Dave**: I agree with graydon
  * **Patrick**: ok
(consensus)
  * **Graydon**: good luck figuring out how to do log
  * **Patrick**: why hard?
  * **Graydon**: flags
  * **Patrick**: syntax extension, probably not macro
  * **Graydon**: introducing syntactic elements that don't exist in language; we don't have mutable globals
  * **Patrick**: interesting. we do have const in foreign mods; that lets you have unsafe reference to external; all we need is some way to define unsafe definition of an external
  * **Brian**: const gives you mutable reference?!
  * **Patrick**: immutable one
  * **John**: what is the feature involved here?
  * **Graydon**: global variables
  * **John**: I'll just read up
  * **Graydon**: logging is currently controlled by global variables injected by compiler
  * **Patrick**: we can have the global variable in the runtime, write it in .c pending figuring out how to do it in Rust
  * **Graydon**: but it's global per-module
  * **Graydon**: this is why we've been blocked on this forever
  * **Patrick**: I think we're gonna need to have unsafe globals for interacting with C code anyway
  * **Graydon**: mostly agree
  * **Patrick**: we'll figure it out

## conditions in #[cfg()]

  * **Patrick**: only have `or` right now; would be nice to have conjunction and negation
  * **Patrick**: I propose: if you specify multiple conditions they're anded, and you can say `not(--)`
  * **Patrick**: that's minimal. we could have and and or operators
  * **Graydon**: comma-separated seems fine
  * **Patrick**: basically gives you disjunctive normal form
  * **Brian**: agree

## function unsafe

  * **Patrick**: diff between `fn unsafe` and `unsafe fn` -- originally we were like what?
  * **Patrick**: general chatter was this wasn't great
  * **Brian**: I tried to remove it once, didn't like it
  * **Brian**: wouldn't mind removing it though
  * **Patrick**: I never write `fn unsafe`
  * **Graydon**: I'd be in favor of removing
  * **Patrick**: just a form that only exists in statement or expression position
  * **Brian**: not gonna lose any sleep if you remove it
  * **Patrick**: sympathize with brevity, but I worry this is the one form that needs to be unambiguous
(consensus)

## trait bounds

  * **Patrick**: currently allow in weird places like structs
  * **Patrick**: don't enforce them there
  * **Niko**: we do, probably not correctly, but we try to
  * **Patrick**: feel like having a trait bound in data (structs, enums, etc) is kind of weird overlap of functionality of trait bounds in code (fn, impl)
  * **Brian**: can't use them till you call a function
  * **Patrick**: don't really serve any purpose
  * **Niko**: can't ever construct an instance that's not e.g. Eq-able
  * **Niko**: when we had class syntax they made more sense
  * **Brian**: user could create your data type, violating some of your intended invariants
  * **Patrick**: if you export the constructor
  * **Patrick**: if not, you control it
  * **Niko**: if you export the type it has a literal form
  * **Niko**: we had this discussion once before I thought
  * **Graydon**: feel like we've discussed this
  * **Niko**: if we ever want them we can put them back
  * **Graydon**: no one's gonna put anything there in the syntax anyway, totally future-proof

## unsafe pointer indexing

  * **Patrick**: ability to index into unsafe pointers -- people want this
  * **Brian**: never felt need for it; `offset` method gives you offset
  * **Niko**: no strong opinion
  * **Patrick**: maybe let's not worry about it
  * **Graydon**: why can't use index method?
  * **Patrick**: unsafe; `index` method doesn't have unsafe bound on it
  * **Niko**: you have to build it in if you want it
  * **Niko**: can certainly support `+` and arithmetic
  * **Niko**: I don't really care
  * **Graydon**: discourages people from doing it, which is good
  * **Patrick**: okay, let's not do it

## u8 * const

  * **Patrick**: every other day someone comes in and asks how to create a fixed-length vector with a const
  * **Patrick**: concerned about cycles: consts are type-annotated, so creates weird cycle between type resolution and const evaluation
  * **Patrick**: given thought in different ways. way I like best: remove field projection from const evaluation language, then nice bottom-up phenomenon: means you can evaluate all the numeric constants without evaluating any other constants
  * **Niko**: do we have array-indexing?
  * **Patrick**: have to remove that too if so
  * **Dave**: still have to do cycle detection between const variable declarations, right?
  * **Patrick**: yes, already do. const-and-type cycles is the hard one
  * **Niko**: I'm in favor of this
  * **Graydon**: I'm in favor of this, if you talk about removing those two operators from the integer constants
  * **Graydon**: if you look at work in constants, there's three expression grammars: full, const, and int const expressions -- latter are ones that can serve as enumerator values; they're the ones that should be here as well; array sizes should only be int const expressions
  * **Niko**: not just property of expression that appears in that position, but any variables referenced by that expression, right?
  * **Graydon**: yes, it's a deep property
  * **Patrick**: this exists today?
  * **Graydon**: believe if you look at constant classification code
  * **Niko**: guess that's a compromise: evaluate those constants that appear in those positions early... (cut off)
  * **Patrick**: all consts that have scalar values, just make them always integer?
  * **Niko**: didn't think that's what he's saying. think he's saying const of some types can use field projection, other types not
  * **Niko**: seems weird
  * **Dave**: not clear what Graydon means, let's let him explain
G:
  * **Niko**: Four examples:

```rust
    // Example 1:
    const foo: uint = ...;
    let x: [int * foo] = ...;'

    // Example 2:
    struct S { f: uint }
    const foo: S = ...;
    let x: [int * foo.f] = ...;'

    // Example 3:
    struct S { f: uint }
    const foo: S = ...;
    const bar: uint = foo.f;
    let x: [int * bar] = ...;'

    // Example 4:
    struct S { f: uint }
    const foo: S = ...;
    const bar: uint = foo.f;
    /* bar is never used in a type context */

    // Example 5:
    struct S { f: uint }
    const foo: S = ...;
    const bar: S = S { f: foo.f };
    
    // Example 6:
    struct S { f: uint, g: [u8 * bar] }
    const foo: S = S { f: bar, g: [ 1, 2, 3, 4, 5 ] };
    const bar: uint = foo.g[0];
```
    
  * **Graydon**: all I'm saying is int constant category is what we should use in array bounds
  * **Niko**: seems to me that's insufficient. have these 4 examples; what I'm concerned about is... sounds like bad case is ex 3: expression is simple, but definition of `bar` requires a field projection
  * **Niko**: difference between that and ex 4 is constant is never used in a type context; patrick said maybe all uints should be simple, but that's pretty restrictive
  * **Dave**: concerned the distinction between simple/non-simple expressions will be hard to understand, and the errors will be hard to diagnose
  * **Graydon**: why does turning off subsections of grammar make the problem any easier? you still have to check for cycles, right?
  * **Patrick**: but then you have to check for cycles between constants and types
  * **Graydon**: but there's no way for a constant to be defined in terms of a type
  * **Patrick**: but constants have to be type-annotated; type checker currently works by doing type collection first -- that now has to be intertwined with constant-evaluation
  * **Dave**: oh, cycle example: `const i: [ u8 * i ] = ...;`
  * **Niko**: we can detect these cyclic conditions, but I think what patrick's objecting to is big phase that does types and constants altogether
  * **Niko**: in patrick's scheme they wouldn't be intertwined
  * **Graydon**: well, int constants
  * **Niko**: but int constants can refer to arbitrary types
  * **Niko**: if you remove field projection, int constants only ever refer to int constants
  * **Graydon**: I'm fine with that restriction in int constants, which are the only thing involved in this early pass
  * **Niko**: seems weird to me to have different legal expressions based on type of constant
  * **Graydon**: no, it's already true. because `&` is legal here. int constants get put in memory in a place
  * **Niko**: don't understand
  * **Graydon**: every constant that is not an integer constant is semantically and importantly semantically different. not just a value, but a place in memory. has an address that can be taken
  * **Niko**: that's not necessary
  * **Graydon**: it *is* necessary
  * **Niko**: why? could take address of any constant
  * **Graydon**: couldn't use in an integer constant in a type
  * **Graydon**: can cast pointers to integers
  * **Niko**: don't think we allow that in safe code, do we? we shouldn't
  * **Niko**: don't think you can cast `&T`
  * **Graydon**: uncomfortable pretending addresses don't exist
  * **Niko**: can find out what they are through unsafe code. it's unsafe if we go to moving GC, for example
  * **Niko**: so you'd be happier if constants with integral type are special category
  * **Dave**: if there are important semantic distinctions, there should be important syntactic distinctions to make that clearer
  * **Patrick**: not totally convinced of utility of field projection in constants anyway
  * **Niko**: not convinced about field projections. seems like you could pull out that value...
  * **John**: error message is gonna be unfriendly: b/c this thing turned out to be int, not allowed to do field projections
  * **Graydon**: this is completely normal in C-family languages
  * **Graydon**: they have `sizeof`
  * **Patrick**: that's not a field projection
  * **Graydon**: same effect, creates backwards projection with cycles
  * **John**: doesn't that argue for Dave's scheme?
  * **Niko**: argument we should just do the combined phase.
  * **Dave**: if we do that, don't need the syntactic distinction as much
  * **Patrick**: maybe we should just combine
  * **Niko**: not so sure. would also require that we type check constant expressions, b/c we have to be able to evaluate them.
  * **Niko**: right now: first we create the types, then can type check
  * **Dave**: could sort of do "dynamic typing" -- check as you const-evaluate
  * **Niko**: seems painful, anyway
  * **Patrick**: could create graph of types and topologically sort, that gives you an evaluation order
  * **Dave**: this is likely to get over-complicated
  * **Graydon**: lemme play devil's advocate against my own position. we've been conservative b/c it's hard to decide. one reason we've resisted `sizeof` so long -- it's junky. target-dependent. and it's not even enough; you end up with `alignof`... we have difficulty with native C libraries that present partial types or whose size depends on size of other types; don't think those patterns are gonna go away.
  * **Graydon**: anyway I agree this complicates the type checker
  * **Patrick**: C doesn't have cyclic dependencies so it's easier for them
  * **Niko**: do we use field projection in const-expressions?
  * **Patrick**: problem is `sizeof` really
  * **Niko**: not so bad b/c you know what its type is
  * **Patrick**: no you don't
  * **Niko**: you can scan for `sizeof` in AST and find it; induces a dependency, yes, but I don't think it's... not same thing. doesn't require you have full...
  * **Graydon**: seems like a chat with community might help, background research on other languages
  * **Graydon**: don't have quite enough to decide right now
  * **Patrick**: ok, that's fine
  * **Graydon**: I'm not wedded to field projection, but it's a bigger problem
  * **Patrick**: that's completely right, `sizeof` is also important to consider

## removal of both type and trait impls

  * **Patrick**: if you're in same module as `T` and you write `impl T : tra` then methods in that impl are available in both the type and the trait, but not if you're in a different module
  * **Patrick**: this behavior is surprising, will probably lead to name conflicts
  * **Patrick**: would like to remove it
  * **Brian**: agree
  * **Graydon**: agree
  * **Niko**: agree

## Eq and Ord reform

  * **Patrick**: don't quite know what to do
  * **Patrick**: design space
  * **Patrick**: currently `Eq` and `Ord` are good for partial but not so good for total. deriving `Ord` is not particularly efficient since `lessThan` and `eq` are different methods.
  * **Niko**: I think the efficiency question is not even necessary. correctness of hashtables is based on...
  * **Patrick**: compare the efficiency of these two approaches:

```rust
    // C++-like Ord
    impl<T:Ord,U:Ord> (T,U) : Ord {
        fn lt(&self, other: &(T,U)) -> bool {
            if fst(self) < fst(other) {
                true
            } else if fst(self) > fst(other) {
                // two comparisons between fst(self) and fst(other) :(
                false
            } else {
                snd(self) < snd(other)
            }
        }
    }
    
    // Haskell-like Ord (also Java)
    impl<T:Ord,U:Ord> (T,U) : Ord {
        fn ord(&self, other: &(T,U)) -> Ordering {
            match fst(self).ord(fst(other)) {        // one comparison
                Less => Less,
                Greater => Greater,
                Equal => snd(self).ord(snd(other))
            }
        }
    }
```

  * **Patrick**: correctness of hashtables: if you put NaNs as key in hashtable, you're going to have pain
  * **Dave**: haha, dealing with this in JS too
  * **Patrick**: because NaN != NaN
  * **Patrick**: fundamental issue is that there's partial vs total, two different interfaces
  * **Patrick**: hash tables and binary search trees want total equality and total ordering, respectively
  * **Patrick**: for ordinary arithmetic operators, you typically want the partial equality and partial ordering
  * **Brian**: why do you want that?
  * **Patrick**: 1) expectation based on IEEE, 2) efficiency in hardware
  * **Dave**: the hardware point is the compelling one, trumps everything
  * **Patrick**: C++ says putting NaN in hashtable is undefined behavior; always use partial equality
  * **Patrick**: Haskell says: we are not IEEE 754, everything is total
  * **Patrick**: as a result, not sure they can use most efficient fp hardware
  * **Patrick**: neither seems particularly ideal. feel that both are suboptimal
  * **Dave**: sounds like you want 4 interfaces; worry about complexity
  * **Patrick**: other problem: what do the operators mean
  * **Patrick**: derive PartialEq and TotalEq, one for operator, one for hashtable; not great
  * **Patrick**: also can't use operator for the impl of hashtable, could be slow
  * **Graydon**: I'm not following. why does hashtable want total equality? why is it undefined in C++?
  * **Patrick**: if you put NaN in, can't get it out again
  * **Graydon**: what does a normal C++ library actually do?
  * **Patrick**: don't know
  * **Niko**: depends very much on precise way they wrote that loop (e.g., whether they use == or !=)
  * **Graydon**: when you say partial equality, you mean one where they can both be false?
  * **Niko**: yeah
  * **John**: wait, does `NaN != NaN` return true or false?
  * **Dave**: true
  * **Dave**: I tend to think that the nature of floating point is that it's hazardous
  * **Patrick**: not so simple because of example code above
  * **Patrick**: wait, for binary search trees, you really want the `ord` method, not less-than
  * **Patrick**: proposal: Three traits:
    * Eq is a partial ordering.
    * Cmp gives you the operators <, <=, >=, and >, and is partial.
    * Ord is the "ord" function and is a total ordering.
  * **Brian**: I don't understand why this is better if hashtables are still broken for floats
  * **Patrick**: here's why I like this: < is not what you want for binary search trees; they want `ord`. and `ord` can be efficiently implemented for substructures
  * **Brian**: so regardless of float problem want `ord`
  * **Niko**: arguably could use `ord` for hashtables
  * **Niko**: float would be instance of `Ord` using Java ordering
  * **Brian**: because `Ord` provides `Eq`, doesn't matter if `Eq` is total
  * **Patrick**: I'm not sure about solving for hashtables
  * **Brian**: thought that was whole motivation
  * **Patrick**: there are many. biggest is that I can't implement `deriving Ord` efficiently
  * **Brian**: okay. only matters for hashtables. everyone else uses `Cmp`. those are still gonna be same thing as always. where does this matter? who will use `Ord`?
  * **Patrick**: binary search trees
  * **John**: current proposal is hash requires `Eq` or `Ord`?
  * **Patrick**: prefer `Eq` and don't care about NaNs
  * **Niko**: me too
  * **John**: could be security issue. could get back somebody else's value if you ask for NaN
  * **Niko**: how?
  * **John**: no, I take it back
  * **Dave**: would urge us to have very clear, carefully written documentation clearly explaining when to use `Cmp` vs when to use `Ord` -- otherwise people will find it confusing
  * **Dave**: should `Ord` <: `Cmp`? since they're related
  * **Patrick**: no because `ord` doesn't use less-than
  * **Niko**: hm, unclear... they could
  * **Patrick**: this solution doesn't solve float at all
  * **Patrick**: sort of hackishly solves floats with `Ord`
  * **Patrick**: if we really wanted to solve it, we'd have `TotalEq` and `PartialEq`
  * **Dave**: haven't understood the deriving part of this issue
  * **Patrick**: want deriving to generate optimal code, but I can't without `Ord`
  * **Dave**: sounds like you should give this some more thought and present it next week
  * **Niko**: or write it up. I think this is the most plausible solution
  * **Dave**: no relationship between `Cmp` and `Ord`?
  * **Niko**: yes. reason is float -- have both a partial and total ordering, so can't just have one trait hierarchy
  * **Dave**: really think we should have a well-written rationale as well as clear guidelines for use
  * **Niko**: yeah
