# Agenda

- SIMD syntax instead of attributes (samsung)
- Rust ARM buildbot (samsung, brson)
- Removing trailing nulls from strings. #8296 (brson)
- order of extern mod, mod, use. (cmr)
- conditions (graydon)

# Attending

graydon, jack, kmc, brson, tjc, dherman, jld, sully, toddaaro

# SIMD syntax

- graydon: i can summarize. as i understood it, structural types have the advantage of being anonymous. anytime you use them it means the same thing. i think at various times people have proposed using `<>` tuples to mark SIMD types, and i've resisted this. i feel like we can get by with intrinsic types. 
- dherman: we came a long road from structural types to nominal types. it's extremely tricky to get right.
- brson: it's not just samsung. the last time someone brought this up in a blog post

newer: http://blog.aventine.se/post/55669497784/my-vision-for-rust-simd  
older: https://github.com/mozilla/rust/wiki/Lib-simd  

- graydon: jens points out that nominal tuples don't work cross crate right now.

- graydon: We had a number of conversations, myself, Jens, Seo and a few others, ending up on a consensus (I though!) on IRC to use nominal tuple types with special lang-item status. Links to the conversation:

https://botbot.me/mozilla/rust/msg/4615603/  
https://botbot.me/mozilla/rust/msg/4616169/  
https://botbot.me/mozilla/rust/msg/4615944/  

That is, we'd define (in the stdlib) a bunch of types like:

  #[lang="vec4"] struct Vec4<T>(T,T,T,T);
  #[lang="vec8"] struct Vec8<T>(T,T,T,T,T,T,T,T);
  #[lang="vec16"] struct Vec16<T>(T,T,T,T,T,T,T,T,T,T,T,T,T,T,T,T);

and the compiler would SIMD-ify uses of these tuples, but no others.

This is similar to the technique originally suggested back in May 3:

https://github.com/mozilla/rust/pull/5841#issuecomment-17107698

Then implemented in

https://github.com/mozilla/rust/pull/6214
and
https://github.com/mozilla/rust/pull/7705

Except extended from the use of a #[simd] attribute to use of a #[lang] attribute. By making the types lang items, we ensure that there is no risk of multiple people defining redundant nominal-but-incompatible SIMD vector types; by making them nominal we permit attaching an attribute to them such as #[simd] or #[lang] in the first place (attributes cannot be attached to structural types like tuples).

This work is presently blocked on compiler bugs: https://github.com/mozilla/rust/issues/7899

# ARM buildbot

- brson: i can summarize where we are. i have an ec2 image set up and buildbots set up. the build scripts are set up too. the only problem is there is a bug that (makes the world blow up) whenever I turn it on. now that graydon's back we should be able to get that fixed soon.

# trailing nulls

- brson: there's PR open from eric that removes trailing nulls and i want to make sure everyone is aware of this before we hit the go button.
- graydon: it's ok for me.
- kmc: does this mean that as_c_str involves copying now?
- brson: i believe as_c_str is gone and there are new methods that involve copying strings
- brson: graydon are you comfortable with this?
- graydon: i don't love it but this was too clever for its own good.
- kmc: are people stubbing their toes on string nulls?
- graydon: anytime you interoperate with a slice you have to know if it's a vector or string slice because they are encoding with one byte longer than they actual are. and if you want to make a string slice out of a buffer you can't do that. you end up going one byte too far. and the user has to guess that, so everytime they use the api they use it wrong. i think if we have a temporary space api then it won't be that bad.

# ordering of extern, mod, use

- tjc: i thought this got discussed before the change, and there were good reasons.
graydon: it's because mod is an item. it's the same as any other item. gareth makes the point that this is a definition.
- jack: definitions shadow use, so that's why they need to come after.
- graydon: it's confusing becuase they think of use as a file local activity so they think those should come first. i think the shadowing argument is sort of the winner here. i'll post to the list to clarify that.

# conditions

- graydon: i've been getting feedback that they are unsuable because they dont' work cross crate. conditions now *do* work cross crate and i'm in the process of writing the condition tutorial now. we need to refine this and would like to get feedback. writing in the result monad is not a feature we want to be foisting on users.
- dherman: this is probably a conversation we'd want to have patrick here for


