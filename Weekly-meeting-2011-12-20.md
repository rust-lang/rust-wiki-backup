## Attending

Brian, Dave, Niko, Marijn

## Status

* Marijn: static portion of type classes working; dynamic portion will probably be quite a bit of work
* Niko: unique closures: got distracted by some syntax work
* Niko: want to generalize the Ruby-like block shorthand to allow other kinds of functions, too; only works when unambiguous (always works in function calls)
* Brian: calling the syntax "block" doesn't make sense anymore then
* Niko: yeah, definitely should get the terminology right. maybe "stack closures" or something

## Varieties of function

* Niko: I'd like to pare down the multiplicity of function types
* Marijn: get rid of bare functions?
* Niko: we currently have five varieties! native, bare, unique, stack, shared
* Niko: I was thinking about whether it's possible to eliminate bare
* Marijn: could just extract raw pointer in stdlib
* Niko: and conversely, a C function `call_rust_closure`
* Dave: what's `native` used for?
* Niko: ABI is different, but I'd like to eliminate it
* Brian: well, we have these wrappers
* Marijn: when you have a known direct call, just use the right ABI; otherwise auto-wrap
* Niko: that could complicate trans... or does LLVM inline the wrapper? that seems like the simplest
* Niko: same possibility for bare functions

## Semicolons

* Marijn: Re: generalized blocks --- what about outside of call? syntax ambiguities?
* Niko: yeah, surprisingly hard. machinery for knowing at parse-time if you're in an "ignored-result" context, but you can't do that anymore; trying to push this logic into the typechecker
* Niko: my patch also eliminates significance of trailing semicolons
* Marijn: seems radical enough to be worth discussing on list
* Niko: I'll get it working and then send to the list
* Niko: could go whole-hog like Ocaml with the `ignore` thing
* Dave: yuck, way too strict

## C stacks

* Marijn: do we have a way to call native functions without stack switch, if we know it will fit within the red zone?
* Niko: inclined to wait till we have data on perf --- it's yet another possibility to add crashes
* Brian: agreed
* Marijn: don't we need a lock to switch?
* Niko: no, it's pretty cheap. just some memory shuffling
