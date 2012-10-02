# Agenda

- 0.4 progress / status (graydon)
- default-move on non-IC types (niko)

# 0.4 Progress / Status

- tjc: I'm working on removing uses of explicit copy mode
- pcwalton: some weirdness with runtime calls which currently has a wacky mode
- graydon: in that case, a run through the docs is in order
- graydon: I am still in discussion with legal with regard to licensing issues

# Camel Case Warning On By Default?

- brson: erickt doesn't like having it on by default, doesn't play well with native bindings
- brson: I would expect you to turn it off explicitly
- pcwalton: I thought that warning would be temporary
- brson: it seemed like a nice thing to push people in the right direction
- graydon: maybe we should just weaken the warning to indicate if it starts with a cap
- brson: I think if the rule is that types should start with a capital letter, it's too weak to bother with.  It's not the full convention.
- pcwalton: I could see separating full-camel-case from the first letter
- nmatsakis: just feels like a lot of modes.  but I don't really care.
- brson: maybe we should just turn it off.  it's not important.
- graydon: it seems like as long as the core/std follow it, it's probably enough cultural precendet
- brson: I'll just turn it off for now. 

# `main()`

- pcwalton: I think the easiest way to bootstrap, if nothing else, is `sys::get_args()`
- pcwalton: I'm in favor of moving the argument list out of the signature of main()
- pcwalton: on the grounds that it removes overloading from the language
- pcwalton: and it doesn't significantly alter the amount of "environmental state" that is available
- tjc: it seems confusing that having an `args` argument might vary from `sys::get_args()`
- brson: on the other hand, I prefer the args vector because you can treat `main()` like any normal fn and don't have to worry about global state
- graydon: weak preference in favor of the argument vector
- nmatsakis: me three
- conclusion: for now, we'll just do the `getargs()` thing.

# Move-by-default

- nmatsakis: http://smallcultfollowing.com/babysteps/blog/2012/10/01/moves-based-on-type/
- graydon: implicitness makes me somewhat nervous
- brson: what does it allow you to remove?
- nmatsakis:
    - capture clauses
    - move, copy keywords
    - move, copy bindings
    - implicitly copyable warnings
    - last-use
- brson: removing all those things is compelling
- graydon: yeah let's think it over
