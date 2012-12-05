# Agenda

- Triage for 0.5?

# Things to discuss with full team

- C functions being designated unsafe (by default, at least)
- add_assign, sub_assign (overriding `+=` etc)
- `&mut [T]` also `@mut [T]` (this one we can just go ahead with)
- empty module scope by default
- unsafe pointer indexing `x[5]`
    - would have to be built into the language
- extending the cfg language
    - `#[cfg(a, b, c)]` should `a && b && c`
    - `#[cfg(not(a))]` as well
    - Supports the full "or-of-ands" normal form
- borrow check changes proposed in [Niko's blog post][nbp]
[nbp]: http://www.smallcultfollowing.com/babysteps/blog/2012/11/18/imagine-never-hearing-the-phrase-aliasable/
    
# Triage

- 281 open issues!
