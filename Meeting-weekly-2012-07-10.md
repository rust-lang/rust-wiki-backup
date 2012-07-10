# Rust Weekly Meeting 2012-07-10

## In attendance:

- graydon
- tjc
- bblum, lkuper, sully, elliott, niko, brian, eholk, pauls

## Agenda:

- Release soon / next day or two
- Default branch = incoming?
- Not doing the pointer-sigil change
- Hygiene

## HR Notes

- FYI, lkuper and sully going to be gone the 16th-28th (doesn't need to be an agenda item, really; this is just the easiest way to let everyone know :) )
- brson will be on vacation next week and a half-ish
- tjc gone week of july 31

## Idle chit chat

- macros are kind of in but only in very experimental form

## Release soon / next day or two

- Graydon: everything keeps bouncing in and out... but I think that everything we wanted for 0.3 is now in.  Some reflection code still to come but it's not crucial.
- Graydon: is there anything that must go in which is not in yet?
- *crickets*
- Graydon: I will have to disappear into release engineering (S3, buildbot, etc).  I am going to have to ask everyone to stop pushing stop for a while because automation gets overwhelmed and we don't have a way to make it stop building.
- Graydon: Now is a good time to do Fun Experimental Background Work (tm).

## Default branch = incoming?

- Graydon: someone (Eric?) was filing a pull request the other day against the wrong branch, and the question came up, "Why does github point us to master if we should be working against incoming?"
    - There is a setting called the default branch which is also reflected in git
    - It's the one that gets displayed on UI, etc
    - I changed that to incoming
    - Unfortunately you can only select one default, not default-per-task
    - Does this seem like a bad idea? Users are thus more likely to get a broken tree.
- tjc: Can you print out a message when pulling?
- lkuper: we can just put it in the instructions
- graydon: I don't know if we can set those type of hooks in github
- nmatsakis: I kind of feel like if you just pull... well...
- graydon: but maybe we ought to be putting our best foot forward in that case?
- tjc: "customer is always right"
- brson: I kind of agree it's punishing casual users for the few of us
- graydon: delta between master and incoming is only supposed to be 1 day anyhow though we have been slack at times
- graydon: I think I'll just set it back for now 
- graydon: maybe just educate those who do have pull requests to target them appropriately
- conclusion: will reset default to master

## Not doing pointer sigil change

- graydon: per discussion on IRC towards end of day yesterday, we discussed changing pointer sigils.  Change was motivated by a problem in pattern matching, we wanted to differentiate between by-ref taking.
- graydon: Short story: keep sigils same and muck around with pattern syntax instead.
- graydon: & == borrowed ptr, * == unsafe ptr

## Hygiene

- pauls: now that token tree stuff has landed want to work on hygiene
- pauls: big things are hygiene, import/export
- pauls: hygiene seems like something you don't want to put off forever
- pauls: going to have to do some research into how the algorithms work, dherman has some ideas for this
- pauls: this will probably involve changing representation for identifiers, which is currently a boxed string
- pauls: typical algorithms need some marks etc which may be stored in a side-table or inline
- brson: we have wanted to intern identifiers anyhow for performance reasons
- graydon: we actually intern them in lexing and then un-intern them... for some strange reason
- nmatsakis: it seems like side-tables are more consistent
- pauls: regardless, it will probably make things longer because error messages tend to mention identifiers and some additional characters will be needing to convert to strings
- graydon: also let's get rid of the old macro system, nothing worse than two systems
- pauls: biggest problem is the *uses* of macros, every #debug and #fmt changes to debug! and fmt!
- graydon: perl?
- pauls: also will want to change the delimeters from [] to {} I believe, not possible with regular expressions
- pauls: proposal is `ident!( expr* ){tt}` where the parenthesized exprs and the token trees are optional, so fmt would become `fmt!("foo", bar, zed)`
- pauls: this allows for three forms
    - function-call like: `fmt!(...)`
    - exotic: `let!{...}`
    - block-like: `for!(...) { ... }`
- graydon: feels to me like simplifying expression list all the way down to token trees with a comma-separated expression list as a particular pattern... it doesn't feel like it's that important to me to have a form that parses expressions ahead of time vs the complexity of multiple invocation forms, we could then allow arbitrary delimeters (`fmt![]` etc)
- pauls: the simplicity of that appeals to me
- eholk: current system allows () or []
- graydon: this seems undesirable
- pauls: was intended to be a temporary thing
- graydon: best seems to me to be that each macro has a particular delimeter chosen by the macro author, just from a simplicity perspective of explaining this to the user
- graydon: operates like CPP or s-expression reader
- pauls: we could do that, it would be possible to change macros over to TT and potentially add ()s an option later
- pauls: might produce additional ambiguities, but it seems rare
- graydon: syntactic ambiguities ...?
- pauls: in `if foo!() {}`, is the `{}` the body?
- graydon: no, just one argument list.
- pauls: in some cases it'd be nice to be able to have an identifier outside of the curly braces for item position macros.  So we might find that there are invocation forms where multiple arguments are helpful. (ed: `proto! foo {...}`)
- pauls: we can table this for later though
- graydon: except that you're in the process of rewriting everything
- pauls: we can put them in braces for now and then decide whether we want parens etc later
- pauls: I feel like the design space is too complex for the meeting
- graydon; you and I can talk about that more, yeah
- brson: let's not do that for 0.3
