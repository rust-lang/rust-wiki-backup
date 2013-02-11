# Agenda:

* 0.6 milestone review
* mut fields, mut vectors, ~mut
* region syntax
* bors (graydon)
* library organization group(s)?
* strategies for bug triage?

# Attending

- A: Azita
- N: nmatsakis
- P: pcwalton
- JC: john clements
- G: graydon
- B: brian
- F: Felix (late and silent)
- T: Tim (late)

# 0.6 Milestone Review

- N: There is this google doc that reflects what we said at the last meeting
- N: but it's out of date
- N: we should try to keep it up to date
- G: I am a little concerned about the visibility issues of a google doc
- G: in particular I think the milestone stuff will eventually involve outside contributors and so forth
- B: I agree, nothing private here
- N: that makes sense, maybe we should just do a wiki page
- P: spreadsheets are not great for planning anyhow
- N: do we have a date for 0.6 in mind?
- P: End of March, right?
- P: I want 0.6 to be "backwards compatible" with respect to the language as much as possible
- P: So I am focused on removing things we plan to remove and/or adding legacy flags
- P: Everyone wants it to be backwards compatible
- B: Can we test this?
- P: I...don't know how, usually we know when a language feature is backwards incompatible, right?
- B: I don't know if that's true or not
- G: I think there's a difference between formally measuring backwards incompatibility---for example, the grammar is only semi-defined, some of the inference algorithms may change
- G: Our current spec is "this compiler accepts it" which is the normal starting point
- G: I don't think we'll be able to promise a lot more than that
- G: We can promise that our *intent* is backwards compatbility
- B: I agree I just don't want to claim stability that we don't actually have
- G: I think clarifying a "statement of intent" with caveats is ok
- Examples: resolve, lifetime scopes, precise type inference algorithm, trait resolution
- G: Can we think about the word "beta"?  Like, when we claim it's "feaure complete"?
- P: I don't want to claim feature complete, just backwards compatible
- P: I would think that when the scheduler rewrite is done, working I/O, etc I would be willing to call it beta
- G: Not to mention working GC
- P: I would say GC and I/O are the beta criteria, at least in my mind, along with a roughly stable language
- G: I'd like there to be a formal grammar
- JC: I have a lot of interest in that though I'm trying hard not to get side-tracked
- JC: I think that in the next couple of weeks it should be possible to put together a token-based macro expander.  I implemented an EBML parser so that we can perhaps put together regression suites over the parser (check that one grammar is generating the same AST as we do today).
- JC: I'd like to spend some time on that in 0.6
- P: What language is this EBML parser in?
- JC: Racket
- N: I am afraid of trumpeting backwards compatibility too loudly
- P: What are some of the things that you are afraid of?
- N: One example would be defaults in region syntax
- P: We can make things more explicit for 0.6 but then add defaults later
- N: Sure but maybe we want those defaults
- P: There is also the opt-in strategy
- P: I don't know if that works with lifetime syntax exactly
- N: We should certainly scour our minds to make a list of what we might want to change
- P: I've been doing that over the past few weeks
- N: which is great, we should do it more formally before we make some sort of statement
- G: we might be able to make a weak-ish statement
- N: I like the idea of having some sort of grid that shows the different areas of the language that are stable and for those that are not yet when we expect it to be finalized
- G: I kind of want to put this aside to do some reading into how people talk about language stabilization
- G: I think we're all violently agreeing and disagreeing without precise terminology and I do think this is important
- G: This is all related to documenting the language and having a specification
- G: Our current reference manual is very ad-hoc (ed: and out of date) 

# Milestone review

- A: I see that tjc is not here but last week we were talking about the 0.6 milestones and some people were saying that the things are somewhat out of date
- N: one thing we were talking about is maybe moving this document to a public wiki
- A: I'm not sure why Dave chose a google doc probably because it was very preliminary at that point
- B: it'd be much harder to format it
- N: can you make google docs world readable?
- JC: you can Publish to the Web or something like that
- G: So should we review this list now or have another meeting?
- A: Up to you guys, I think the point is that it'd be good if we could update this document so that it more accurately reflects what's going on, perhaps update the status
- G: I think we should discuss a bit over IRC...
- A: I know tjc mentioned in paricular that his list was out of date
- G: So there are two major philosophies for releases, time-driven and feature-driven
- G: We've been sticking to a time-driven approach: take what's ready and go
- G: Planning documents are interesting from one perspective
- G: Some of these items (e.g., fix ICEs) are not something that can ever be "done"
- A: If we're working on a date-based philosophy, then we should just remove things that won't get done, or substitute higher-priority things
- N: This sort of duplicates the issue tracker, is that the right place to do this?
- G: One of the things I want to talk about is strategies for bug triage
- G: At this point our bug tracker is overwhelmed
- P: I don't even use it anymore, to be perfectly honest
- G: That to me is a major problem
- A: Because there's too much?
- P: I go there like once a week but I can't keep up
- G: I'm not sure if they were ever fully under control but they're definitely not now
- P: I don't know I feel like my priority right now is not fixing misc bugs 
- A: Does it make sense to do triage closer to the release date?
- P: We do but usually a lot gets bumped to the next release
- JC: But it's good to know and it seems valuable to be aware of what's still broken
- A: One hour is not enough to both have discussions and do triage
- P: We should get back to the other things we have to discuss

# mut fields, mut vectors, ~mut

- P: Is everyone in agreement that these should go?
- N: What do you mean by mut vectors?
- N: In particular is this ok: &mut [int], @mut [int]?
- P: Yeah those are ok but not &[mut int] @[mut int]
- N: ok, then yes
- N: There's an open bug about being able to capture mutable variables
- N: The only way to have a ~fn() with mutable, encapsulated state is to use a struct with a mutable field 
- N: I'm not a big fan of capturing mutable variables by copy, though maybe moving in would be ok
- P: I think we have to discuss this outside the meeting there is some design that needs to be hashed out
- N: ok I'll try to sketch out something in a blog post I think I have an idea of a simple extension, we can hack around with cell for now...
- B: ...but the cell is scary because it'll look immutable
- P: I really want mutable fields to not be something people reach to
- B: We could start with some sort of warning and then just allow it in cell
- P: I'd be sad that the borrow checker must still think about it but that might be the way to go
- P: Does everyone understand the difficulties that the borrow checker has with it?
- G: I don't feel like I have a full understanding of this issue
- P: The biggest problem that people encounter is if you have a ~[T] in a mutable field and an & or @ pointer to that mutable field, you get errors, people get burned by this
- G: Yeah, I have hit that too
- N: The issue is that sometimes today you need the option dance to move values through ~fn(), maybe this goes away with once fns
- G: If pcwalton lands what he wants to land, does everything break?
- P: My goal is to remove mutable fields everywhere from the language and make the parser stop parsing them, but maybe that's not realistic
- P: A second goal might be to remove as much as we can and then have a lint mode
- N: The real question, and I'm not 100% sure what the answer is, whether we will need something like mutable fields long term, or whether once fns will suffice
- P: There seems to be some design work that needs to be done but I think the bigger question is whether we have consensus that we should make mutable fields harder to use and more hidden
- G: yes, agreed
- P: ok so action item niko and I will talk over the design issues and I will continue as planned

# Region Syntax

- N: First of all let's review
- N: One thing we decided was whether lifetime parameters must always be specified on types or not

    struct Foo<'lt> { ... }

    fn bar0(x: &Foo) { ... } // legal?

    fn bar1(x: &Foo<'a>) { ... } // or do we have to write this?
    
- N: The other issue which was raised concerned named lifetime

    fn bar1<'a>(x: &Foo<'a>) { ... } // or do we have to write this?

- N: Problem is you cannot specify a value for this parameter today

    bar1::<'a>(...)
    
- G: I introduced this requirement that we have a binder form
- G: I've thought a bit more about it, is it possible to nest?
- N: There is this case:

    &fn<'a>(&fn(&'a))
    &fn(&fn<'a>(&'a))

- G: Are you sure that there is only one sensible meaning here, get rid of the binder
- N: I'm not 100% sure I'm like 99% sure, there is a kind of workaround:

    &fn(&fn(&'a int)) <-- binding on the inner scope
    &fn(&'a (), &fn(&'a int)) <-- force binding to the outer scope
    
- JC: Yuck
- N: I think this is more of a mind-twister than a realistic example, but it could come up
- N: My proposal is the 
- P: (something about omitting the use of the binder...)
- N: I don't understand about omitting the use of the binder

    &fn(x: &int, y: &int)

- N: As long as you don't name the lifetime they are separate and bound to the innermost fn
- N: If you wanted to name it, you would introduce the lifetime names with a binder:

    &fn<'a, 'b>(x: &'a int, y: &'b int)
    
- P: To me the question is what about lifetime parameterized structs, do you have to explicitly qualify what the lifetime is?  Or at least acknowledge the existence?
- N: I think if you don't it's consistent, but only at use sites
- P: But 

    struct Foo<'a> {
        x: &'a int
    }
    
- N: Yes, we just wouldn't allow anonymous lifetimes in a struct
- P: But this is ok?
    
    fn foo(foo: Foo) { … }

- N: Yes.  It'd default to a fresh lifetime.
- P: I argued against it before but you're correct it's consistent and I don't feel strongly about it either way.
- G: I think consistency is important
- P: Having it on the decl at least makes it clear that there are lifetime parameters being introduced
- N: Having a fully explicit syntax also means that a tutorial can introduce using more explicit notation and then drop the extra bits
- P: One unresolved thing, is this how I would write it if I wanted to give an explicit name:

    fn foo<'lt>(foo: Foo<'lt>) { … }

- P: OK, that sounds good to me, I agree, what do other people think?
- tjc: the proposal is that you can make it fully explicit if you want to but otherwise...
- P: ...at use sites, we will apply defaults, but at decl sites you must be explicit
- G: Essentially what we're arguing over is fn signatures, currently we don't do inference on fn signatures, under this proposal we do a tiny amount
- N: Just defaulting, not inference
- G: Yeah more like not requiring `-> ()`
- G: I think that's ok, the binders can get overwhelming otherwise
- N: Another issue I want to make sure people follow relates to impls:

    struct Foo<'lt> { x: &'lt int }
    impl<'lt> Foo<'lt> {
        fn method(&self, v: ...) {
                       // ^^^^ today: short for &'lt Foo<'lt>
                       // I want to make it more consistent with other parameters:
                       // &'foo Foo<'lt>
        }
    }


