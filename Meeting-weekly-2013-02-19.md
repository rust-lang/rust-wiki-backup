## Attending

Patrick, Tim, Brian, Azita, Graydon, Niko, Felix, John

## Agenda

- Closure inference (@||, ~||, etc) (nmatsakis)
- Rustpkg and module / library / binary naming conventions (graydon)
- associated type synonyms? (RFC is github issue 5033); important for library dev? (pnkfelix)
- Legality of whitespace (jclements)
- Capturing mutable variables (nmatsakis)
- Moving from owned fields and so forth (nmatsakis)

## Closure inference

- N: closure inference: wanted to talk about inference of closure types, toss out possibilities. Today, if you write something like this:

    |a, b, c| expr

- N: In theory, we infer whether it's owned, managed, etc. In prinicple works, in actuality no. I plan to improve it, could be better, but: should we make things more explicit? Current situation can be surprising. If we infer stack, upvars are ..., otherwise upvars are ... , could be a significant difference. Sticking point is if you write:
- N: do spawn { ... }
- looks nice, owned closure
- N: for foo.each { ... }
- also looks nice, borrowed closure
- N: In the do/for forms: infer
- N: Otherwise, you would write
    - |x,y| for borrowed
    - @|x,y| for managed
    - ~|x,y| for owned
- N: what do people think? infer only in 'do' and 'for' forms.
- B: why is it okay to infer for only these two?
- N: there's always a function that's going to tell you what type it is, as opposed to the general form, where the inference might be more remote.

     let x = |x, y| ...; // in fact this is @fn...
     x(1, 2);
     foo(x); // ...inferred due to this point in code
    
- N: Seems a little long-distance. The do/for forms are not long-distance.
- B: not that it's easier to infer, but just to read?
- N: sorta, yes. Is this a problem that other people see? Ever had this problem?
- G: whenever it goes wrong, I just write out the closure type.
- P: I find it annoying. I have to write out lengthy ... consult the signature. I agree with Niko
but I'm also ambivalent. Maybe if you use a sigil, you get that sigil, otherwise it's inferred.
- G: +1
- N: also an option
- G: seems consistent with other things
- P: like vectors or strings... ish
- N, G: no. Only integers are like that.
- N: you never infer the type of a vector. the naked form is just a fixed-length...
- N: write with no sigil, get something on the stack.
- P: thought about making literal strings work that way
- N: yes. You could imagine strings doing more inference.
- G: I find 'do spawn' a little to magical to read, wouldn't mind having to write some reminder that it's owned.  Each time, I say "it looks like a borrowed pointer, but it's going to be a ..." 
-N: I think we all thought that was too ugly. Should not have to ...
"... use inference to know if the capture is by reference"
- B: data point: in scheduler, all the async stuff involves owned closures, sneaky how many owned closures you can create without thinking about it.  
- N: usually, sigil for every allocation, but that's not true here.  
- P: good point. we've done that in the past. Do Spawn makes concurrency very natural, which is what we want.
- N: if we made it explicit:

     do spawn ~|| { ... }
     do spawn ~ { ... }?

- N: or not? Do we need the two bars? It would be nice to have a formal grammar here. Not sure whether this is ambiguous.
- G,P,N: unsure....
- P: Could also write:

   do~ spawn { ... }

- N: true....
- P: not sure that's better.

<discuss, discuss.>

- G: syntax bikesheds.
- P: table this?
- N: get it out there before forgetting about it.
- P: I'm in favor of the optional sigil. agreeable to all.
- All: aye.
- N: can be explicit, retain ability to do inference for now.
- N: currently used in do spawn.

## rustpkg

- G: I landed rustpkg, just to get ball rolling, just to avoid continual rebasing. All of the things pointed to by cargo mostly don't build under rustpkg. People are starting to upgrade, adding file called package.rs . Bug, we were going to try to infer more. So, want to talk about problem of inferring package.rs. talked to z0w0 and Brian, read Go code.  Don't want to come up with lots of new names ... (laundry list of names) ...
- ?
-?
- G: One of the nice things about Go is the user interface experienc, where lots its derived from a single identifier. If it's a Source URL, that works, it writes that to disk in a directory that's spelled the same way. a little like the old java system, with fewer components. That's not bad, but it gets a little unclear in repos with more than one buildable thing. If I try to import from one of these things, and only want to build one of the sub-libs... Extern mod questions, if I install: which one do I build?
GThere are solutions that are not too zany, proposing this one: based on conventions of file stems. 
  - no more .rc files is a goal here
distinguishing crate root from non-crate root
distinguishing binaries from libraries, nice if it didn't have to parse it to look for the crate mode attribute (right away). 
  - already have convention item in dir/mod.rs <=> dir::item
  - pkg.rs is what rustpkg looks for when looking for custom build rules
  - proposing: dir/lib.rs => dir is a buildable crate short-name for a library
                    dir/bin.rs => dir is a buildable crate short-name for a binary
- B: Doesn't like the mod.rs convention.
- T: agrees
- N: likes it
- J: you're doing it wrong (emacs settings *could* show the full path)
- N: a lot of ppl use the default for some reason; hence, a good argument
- P: Never been bothered by it in vim

[editor wars]

- N: I suggest `(toggle-uniquify-buffer-names)` for emacs users
- P: node.js has a similar convention, so it's not totally unreasonable
- N: we have a lot of modules, but not too many lib.rs's
- B: except in Servo, where there's 2 dozen, but maybe it's not representative. Agree that pkg.rs would be better than mod.rs

  - option #2 here: foo/foo.rs ==> library-or-binary and you have to parse to find out which
  
- B: Rather than putting something in mod.rs, I prefer to put it in a file w/ the same name as directory
- G: Not bad, but doesn't auto differentiate b/w libs and bins. We tend to parse everything programmatically when installing anyway.
- P: I like that idea. Do we have to force a naming convention if we just parse things?
- G: You don't know which .rs files and which directories to start parsing from. You could parse them all & look for mod directives, put them all in directed graph... not fun. Esp. w/ a large test suite. Don't want to parse all files.
- N: I'd rather be told what to name my file, as a potential user. It's simpler if there are clear instructions. Explaining inference is confusing.
- P: Likes everything
- F: Could give G's semantics to bin-<suffix> or lib-<suffix>
- G: go tends to put named prefixes on things, like a file test-foo is considered a test, bench-foo is a benchmark. Things are binaries if they're injected into a package called <main>.
- P: You can always override this w/ an explicit package.rs, right?
- G: Yes, this is about defaults.
- P: I like the bin and lib thing, and you can override if you don't like it. Convention-over-configuration thing
- N: How often do you think ppl want both? (if ever)
- G: One lib + 1 bin? It's not that common. Usually a package is one or the other.
- N: Advantage of having bin/lib over using the name is just that you can tell immediately w/o parsing anything whether it's a library?
- G: Yeah, you can have both in a subdirectory, without a library and binary fighting over the name. You don't have to nest things in a directory.
- J: I like it.
- G: Nesting things in a dir seems reasonably common; every go package seems to be nested in a dir, weirdly.
- F: I like this better than nesting.
- N: I don't quite understand why you don't have to put it in a dir. Maybe I don't know what you mean by nesting
- G: You can only tell whether something is called foo/foo.rs if you have an enclosing directory called foo
- N: It's going to be taken after the name of the repository?
- G: Yes
- N: So if I clone it into another directory, it'll have a new name?
- G: Yes
- N: So it seems like best practices should be that you have a directory.
- G: Yes, that makes the short name more likely to be stable. A lot of people are naming repositories rust-<mylibrary>, and we don't want the shortname of the library to be that. Maybe sdl/sdl.rs inside the repo is a good way to establish the short name you want?
- J: You could have other stuff in the git repo besides what you want to appear on the user's computer
- G: That's common, the issue is if you point rust-pkg at a repo and it doesn't have specific build instructions in a pkg.rs, what should it do?
- N: If we move to src/src.rs as the convention for modules, we're going to have a lot of matches if we do that, right?
- G: interesting point. Do you have a preference?
- B: I don't, really, seems like everything has pros and cons, nothing's perfect.
- N: I feel like the lib and bin is more declarative, less likely to be accidentally chosen.
- G: Probably, though we actually have a directory inside Rust atm called "lib".
- J: someone should make an executive decision
- G: no firm preferences, I'll post on the mailing list
- B: I want to abstain. Let's do it.
- G: I won't poll the mailing list then, will pick something dictatorially
- B: Re: repo names, do we want to standardize that? The rust- one was picked randomly long ago, and it's not that good. Most languages don't do this.
- G: go programs do it, js programs put .js at the end
- B: We have one other competing distinction, project.rs
- P: I prefer rust- to project.rs
- T: project.rs seems to imply it's just one file
- G: no strong feelings. Would be nice if we don't have to infer the shortname from it.
- P: I feel a little uncomfortable w/ that, yeah. I usually do rust- if it's just an external library binding, come up w/ something more creative if it has a significant proportion of original code. sharedgl, for example, vs. rust-opengl. (original vs. bindings)
- B: That's good. So let's not make any decisions on this topic.
- G: I will essentially pick inference defaults by fiat.

## Associated types

- F: associated types; however, Niko said this isn't our first rodeo. I think this might matter for library developments.
- N: I would rather not discuss it now, it's a complex topic, but it will be important at some point
- P: I think we'll want associated types but not as a Rust 1.0 feature. None of us have time.
- N: Connected to other aspects of traits, could be simulated. Could be verbose b/c you end up with more typarams than wanted. OTOH, the syntax you proposed has limits, requires a different syntax for trait bindings. I'd rather talk about it a little offline, write a blog post. Too early a stage of discussion.

## Legality of whitespace

- J: legality of whitespace. Current situation: paths are made up of a sequence of tokens: ID modsep ID modsep... From later code (macro expander...)'s perspective, the unit *ought* to be a path. (Path = list of IDs.) I claim the world wd be a simpler/nicer place if we closed up our notion of a path, regarded that as being what we now think of as an identifier.
what does this have to do w/ whitespace? Currently, we allow whitespace in the middle of a path: a::  b is a reference to b in the package a. I claim things wd be simpler if that whitespace were disallowed; if the token was bounded by whitespace. It's not the end of the world either way. I grepped existing code and found one position in entire source code where this "feature" is used (where someone wrote a:: b) and it looks like a mistake. Searched for "a ::b", turns up all the time (b/c of things like "return ::c"). You don't want that whitespace to be collapsed, so parser has special cases.
I claim disallowing whitespace in paths would simplify parser, macros, etc.
- N: I think it's strange that we *don't* allow whitespace, we have some weird hacks. B/c of the way we do keywords
- G: that's old code for keywords
- N: We don't do keywords right; that's why it's still needed. They're kind of half-keywords.
- J: Do ppl think it's important/valuable to be able to have whitespace in paths?
- T: no
- N: it's surprising that it can't be done, but I don't know why it's relevant
- J: it can be done, but which is best match for intuition?
- N: It can't always be done. It can be done in both cases
- J: For macro hygiene, macro expander wants to attach info to references, which currently are paths. Great! But these "paths" only appear as disconnected sequences: ID, modsep... So would be nice if token trees used the same representation as ASTs. Could be a matter of doctoring token trees a bit (that would be a reasonable soln)
- N: Suppose I have a macro parameterized by X and I write X::Y in my macro expansion. X is a module name supplied by the user. Does that make sense, in light of what you're saying?
- J: That's such a bad idea
- N: Why? That's exactly what associated types would do
- J: In some sense, I claim it's totally antithetical to macro hygiene.
- P: Not for associated types, N has a good argument. If you don't allow macros to do this--
- J: You'll want to allow macros to do this, but what you want is to say "I'm capturing this binding". The point of macro hygiene is it's totally fine to capture bindings, but you want to say so.
- N: What binding is being captured?
- J: X::B
- N: I want them to give me a module and I capture module:: ... but there'll be 4 or 5 things there
- J: Fine -- in your macro you'll say "capture(...)" and the things you're capturing. Hopefully a small amount of extra work to do, which says "you know, I am capturing this thing"
- N: Doesn't seem like a capture. You'd write something like: 

     macro(M) = 
         Foo = capture of M::foo
         Bar = capture of M::bar
         { (use Foo, use Bar) }
         
         
- J: (misunderstands N)
- N: This wasn't my use case, wanted a prefix of a path or the middle
- J: I don't care, all the same to me
- N: seems v. different. If it's in the middle, I see your reaction. But if it's at the beginning, it's a shorthand for the user to supply me w/ n arguments by only writing 1
- J: Doesn't seem like you should need a macro. 
- N: More concrete examples... Something like uint_template; I know there's just a small # of things it can be parameterized over. I can write a macro parameterized over the things, or one that's parameterized over a module (that contains the things).

     macro(MAX_VALUE, MIN_VALUE, ...) {
         ... MAX_VALUE, MIN_VALUE, ...
     }
     macro(LIMITS: ModuleName) {
        $LIMITS::MAX_VALUE, $LIMITS::MIN_VALUE
     }
     
- N: Offline
- J: This is sort of orthogonal to our orig. discussion. You're asking how hygiene is going to work; I want to answer that too
- N: Doesn't seem totally orthogonal
- J: Nothing to do w/ whether token trees include paths or IDs. It really doesn't. I think I should answer your Q, in a rust-dev email.
- G: An interesting point, I think predicating it on whitespace may be the wrong angle. Think in terms of unifying the concept of "path" and "id"
- J: That's all I was intending to say, yes.

[we get kicked out]
