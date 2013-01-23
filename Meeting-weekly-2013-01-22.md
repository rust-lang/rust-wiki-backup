# Agenda

- composable for loop protocol (nmatsakis)
- remove rule that requires unit type for top-level do, if, etc and replace with warning (nmatsakis)
- limit destructors to owned type (nmatsakis)
- syntax for multiple type bounds (nmatsakis)

## For loop protocol

- N: http://smallcultfollowing.com/babysteps/blog/2013/01/16/revised-for-loop-protocol/
- N: Short vers: argument returns bool, iterator itself returns (). hard to compose
- N: Proposed change: the iterator passes on the bool so that they can be composed
- N: Additionally want 'for' to always return (). reason: top level statements without semicolons must return ()
- (agreement)
- N: Might also want a newtype boolean or enum to indicate break/continue
- (niko: explains 'do')
- B: Newtype easier now with lang items

## Change in rules for block structure (if, do, etc.) return types

- N: Want this as a statement:
```
    if cond { ... } else { ... }
    foo();
```
- N: Also want this:
```
    let x = if cond { ... } else { ... } + 10;
```
- N: We could insist on semicolons but it looks funny.
- N: Today: if the statement begins with `if`, `do`, `for`, a few other such keywords
- N: Parse as a statement, that is, do not continue past closing brace
- N: But this could be wrong:
```    
    fn foo() -> int {
        if cond { ... } else { ... } - 10
    }
```
- N: We won't parse this as intended.  Today, though, we would report an error if the `if` does not have a unit type.
- N: But there is one important case where this breaks down:
```
     do task::spawn { ... }
```
- N: spawn returns the task id but you sometimes don't want it.
- N: ok, maybe not anymore, but it used to
- N: another option: keep current parser but issue a warning is there is a binary operator on the same line as the closing right brace
- B: So we're going to look at the spans?
- N: Yeah, maybe.
- G: not in favour of EOL-dependent algorithm
- N: Graydon, even if it's just a lint warning?
- N: Graydon, what about about if we just look at the next token and don't care about the spans?
- G: more reasonable, yeah. "}" followed by binop.
- N: OK. That's fine with me.
- G: I think that's more-or-less emulating an ASI rule as a lint mode, which seems fair
- N: Yes, that's the idea.  Not actually affecting the parse, but still using ASI-like heuristics to issue a warning when you're probably not getting what you expect
- P: Can we just do warn-unused-result?
- N: Yes
- G: \o/
- N: I do not know how to interpret this emoticon, never have :) oh it means happy
- N: let's discuss this on irc---too hard here!

## Limit destructors to owned type

- N: Graydon said this is what he wanted from the beginning
- B: Includes borrowed pointers?
- N: No borrowed or managed pointers except &static
- G: well, back when the distinctions were drawn along different lines (cyclic vs. acyclic and such), but that dtors have always been limited to a small-ish set of types, those that can safely be destructed top-down w/o potentially referencing nonsense memory. just a question of figuring out what the safe subset is.
- N: Any disagreement?
- *crickets*

## Multiple type bounds

- N: Current syntax uses whitespace as a separator, ambiguous with rooted paths
- N: Propose `<T:Foo+Bar>` as alternative
- N: Graydon?
- G: a bit gnarly but tolerable
- N: Good enough!
- B: We're done here. Thanks
