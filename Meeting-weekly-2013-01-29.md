# Agenda

- "// NOTE"
- Region syntax
- "impl Trait for Type"
- Remove newtype enums
- Copy trait
- Mut fields
- Remove const/pure?
- Ord + Cmp

# NOTE

- You can "// NOTE" in your code and `make check` will print it out

# Region syntax

- N: I wanted to discuss my preferred syntax
- N: Discussion on the mailing list led to one possibility:
  - 'lt
  - &lt/T --> &'lt T
  - T/&lt --> T<'lt>
  - &lt/T/&lt1 --> &'lt T<'lt1>
- N: Advantages I see are:
  - syntax-highlight it
  - explain it: 'lt is a lifetime
  - possibly use it for labels - lifetimes and blocks are similar
- N: Could then manually annotate lifetime of borrows if we wanted
- N: Also rvalue temporaries
- B: Does this preclude us from using ' in identifiers ever?
- P: Only the first one, like a digit
- JC: Does this preclude character constants?
- N/G: No, there is lookahead
-G: mostly agree with one exception: can we do &<'lt>? 
- N: I'm ok with that too
- B: that leads to this: &<'lt> T <'it1>
- G: Yes, but in a way that's more consistent to the user's eye---angle brackets with tick
- P: I mean, it's the lifetime of the `&`
- P: You can kind of read angle brackets as "of"
- P: "array of int", "pointer of lifetime"
- N: Today the defaults mean that "&<'lt> T" is equivalent to "&<'lt> T<'lt>"?
- P: I tend to think no default
- P: Every time I put a borrowed pointer inside a structure I wind up qualifying it
- N: Ok, I don't think it's that common 
- B: But you will see something like &<'lt> Array<Foo>
- N: not that common, but does happen, yes.
- G: Suppose I have a structure that contains a borrowed pointer within
- G: Am I always going to be writing it?
- N: Just like today `&T` --> `&<'lt> T` where 'lt is fresh
- N: It might be nice to have `T<>` --> `T<'lt>` where 'lt is fresh (or something)
- G: Let me write out an example:

```
struct Foo {
    helper1: &Helper<Thing>;
    helper2: &Helper<Other>;
}
```

- N: the declaration would be changed to:

```
struct Foo<'s> {
    helper1: &<'s> Helper<Thing>,
    helper2: &<'s> Helper<Other>
}
```

Here 's would be the intersection of the two lifetimes

- N: How would you use it, that's the other question. Today you can write this:
 
      fn foo(x: &Foo) { ... }

And it expands to:
  
      fn foo(x: &<'lt> Foo<'lt>) { ... }
      
- N : inferred lifetimes are usually the "right thing" (for me, but I know precisely what's happening).
- P : I usually have to add explicit lifetime parameters.
- N: Yeah if you do anything more than read from the value---particularly if you want to return pointers into it
- P: surprising that multiple lifetime parameters on a type infer to the same lifetime
- N: yep, sometimes, the reasoning was that you wouldn't need something that lived longer than the pointer to it, but that's not always correct
- P: My personal feeling is that if you are putting borrowed pointers into structures you have to understand lifetimes at a reasonable level, and there's no way around it
- P: So having defaults won't succeed at sweeping lifetimes under the rug (which was their historical motivation)
- P: I think that works pretty well for by-ref parameters
- P: but it's not working when you put pointers inside structs
- P: but when you start wanting to put borrowed pointers inside structs or return a borrowed pointer, then you start having to know lifetimes, and the defaults are unhelpful in that regard
- N: I find that persuasive
- B: are you clear on the plan?
- N: No
- B: me either
- P: I move that lifetime-parameterized structs must be fully explicit
- B: I think all parameters must be fully explicit
- P: The one case where we should have defaults is for `&T` in a function signature (`&mut T`)
- G: like `foo()` above, you can omit the angle brackets?
- N: the only time you'd need angle brackets is in the event that one of your parameters has a type that is a lifetime-parameterized struct (e.g., `fn foo(x: &Foo<'lt>)`)
- G: If it is a lifetime-parameterized struct, ...
- N: ...basically you have to put a name because you have to supply a value for all parameters, both lifetime and otherwise, on a type.  Sometimes this name will not be important.  
- G: I think I'm ok with this, it's chattier than I wanted, but I think 
- B: in the example that Graydon gave, the lifetime is not declared...
- N: ...I presumed that we wouldn't change this, and not have an explicit binder
- G: yeah, it's a bit odd
- B: this example looks odd b
- P: `fn foo(x: &Foo<>)`
- N: or `foo(x: &Foo<'>)` where `'` means: fresh lifetime name
- N: two questions, a shorthand for lifetime names that aren't used and the other is "do we want to declare all lifetime names"?
- P: I find `&Foo<>` or `&Foo<'>` to be nothing but confusing
- N: I think it's more valuable if we make all binders explicit
- G: Why is that generating fresh names when you don't give any lifetime parameters a problem?
- P: I personally find that whenever I have a lifetime-parameterized struct I want to be reminded that it's lifetime parameterized, it's key to my mental model
- P: Otherwise I get confused by the borrow check errors I see
- JC: In Java, type parameters are always explicitly mentioned
- N: You mean something like this: `fn foo<'lt>(x: &Foo<'lt>) { ... }`
- P: that's pretty chatty in the "read-only" common case
- N: I like the idea of declaring named ones but also having an anonymous form
- G: I agree with that
- B: do you mean that you can put `'lt` or not, Niko?
- N: I meant probably some way to say "there is a lifetime parameter that I am not giving an explicit name", hence `Foo<'>` or Foo<'_> or Foo<_>
- G: Patrick you said you want to be reminded...
- B: Of the names or just that they exist?
- G: Is it so bad if you only have to be reminded when doing non-trivial things?
- P: I won't stand in the way if people don't want to have to declare parameters explicitly
- G:  I'd like two cases:

```
  fn foo(x: &Foo) { ... }   // for trivial uses
  fn foo<'lt>(x: &Foo<'lt>) { ... } // or
  fn foo<'lt>(x: &<'lt> Foo) { ... } // or
  fn foo<'a,'b>(x: &<'a> Foo<'b>) { ... } // or
```
  
- P: I probably wont' write it that way, but I wouldn't stand in the way
- N: I'm happy with that, personally
- P: I worry that the second or third example `(&<'lt> Foo)` can be confusing, because only one of the two parameters is specified
- P: Basically when things are fully inferred I get it but the partial examples seem somewhat confusing because the distinction between the lifetime of the pointer to the struct and the lifetime of the contents of the struct is a potentially confusing distinction
- N: We are sweeping it under much less in this proposal
- G: I think that case 1 is reasonably common in function calls
- G: Currently, if I omit both of those it infers a single lifetime for both
- G: What if it inferred two different lifetimes? Would that cause the errors to go away?
- N: I agree I think it will infer two distinct lifetimes but I don't think that'll make errors go away, well, I guess it really depends on the error.  Most of the things people were complaining about on the list were issue #3148
- P: most of the time I personally get errors it's because I didn't realize that a value was lifetime-parameterized
- G: we've been on this topic a while can I consolidate what we've discussed and leave stuff for the future
- G: I think we've settled on 'lt for referring to lifetimes, &<'lt> etc
- G: we're all ok with more explicit binders and more explicit parameters
- G: the only undecided is whether lifetime-parameterized struct must have its parameters explicitly named inside a function signature
- G: there are various things in flight (defaults, these changes, write barriers) that may affect these results
- P: I agree

# impl trait for type

- P: I would like to decide and settle this syntax question.
- P: I have proposed `impl trait for Type` to more syntactically distinguish trait impls from inherent method impls, I made my case on the mailing list
- G: We may just have to do a vote on this one, it's pure syntax
- B: The only thing that slightly worries is that it closes the door to impls that implement multiple traits. We had that and removed it. 
- N: Couldn't you write `impl T1+T2 for Type` or `impl T1, T2 for Type`?
- B: You could, it's ...  ok.
- G: Can we just vote?
- JC: Abstain, but just to clarify, would there still be an `impl Type : Trait` syntax?
- G: I'm in favor of the colon as is
- B: What is the rational? 
- P: I think that type-and-trait implementations are sufficiently different that we should make them look more different, also methods attach to the first mentioned thing in all cases
- JC: Could you make it clear that it's implementing for a tuple?
- *various grumbles emanating from Vidyo*
- G: this is purely an aesthetic distinction I think
- G: Raise hands for colon: G, TJC
- G: For for: P, N, B
- G: Abstrain: JC
- *general disatisfaction*
- G: `Type : Trait` is used everywhere else, just to clarify, it's a symmetry thing in my mind
- N: I personally find reading `impl Type : Trait` hard, which is why I voted how I voted
- tjc: I also found it confusing initially
- B: the `for` syntax does read well, it's pretty obvious what it means
- G: ok, fine, I have to go in five minutes because I have an interview
- P: let's do `impl trait for type`
- G: ok

# mut fields

- B: Removing mut fields seems dangerous because of unsafe things
- B: that may cause mutation that's not evident in the type system
- P: mutability isn't part of that though, and it's at the fn level
- N: think `restrict` and `const` in ANSI C, eventually we might supply such hints
- P: ok, then just do an attribute










