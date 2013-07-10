## Agenda

- External stacks (nmatsakis)
- Brief summary of macro next steps
- OSCON final count (who's going)

## Attending

dherman, niko, felix, sully, kmc, ecr, toddaaro, azita, tjc, brson, jclements, jack

## External stacks

- N: How to manage split stacks, particularly re: calling C functions. I don't know exactly what's the right behavior. 

    What we do today: whenever you link to a C function, we declare a special Rust wrapper that bridges ABI differences. The wrapper switches to the C stack, which is 2 MB big or something. 

    What we're trying to do: when you link to a C function, you have the real function pointer. No implicit gap. When you call the function, take advantage of P's changes where we tag the function as a called-in-FFI function. When we enter the Rust function that calls the C function, we request a big stack using the morestack mechanism. This has the advantage that you can call C functions by pointer without knowing what they are. We can also statically link to C functions, which plays nicely with hardware prediction/lookahead. Other advantage: user can manually tag callers, so if you call a C function in a loop, you won't have to switch stacks on every loop iteration. This reduces the cost of calling C function.

    One potentially nasty side-effect: suppose the task spawn function calls a C function. Then, as soon as you spawn a new task, you request a 2 MB stack and you lose all the benefits of small stack switching. To reduce costs, want to move stack switching as early as possible, but if you do it at the beginning, you lose the benefits. Currently, if the compiler decides you're calling a C function and it gives you a big stack, you may not realize you're switching. 

- D: The big-stack thing sounds like an effect. How does that work in the face of higher-order functions?

- N: it's not an effect. It could potentially be, but isn't recognized by the type system. Anytime you call a function, it allocates a certain amount of stack. All this does is allocate a lot of stack space. It's not really visible to the caller, so it's not an effect.

- D: Is it possible to decide on answers for any of this without resolving the larger discussion that P started on the mailing list?

- N: The two discussions are intertwined. The thing I don't like about split stack is this mismatch; hard to come up with something that does the right thing if you don't think about it, but also gives you control. Danger of ending up with stacks way bigger than you needed. Other aspect is performance cost.

- J: Every function that does IO is going to get a big stack, under the new scheme?

- N: Any function that directly calls a C function would have a big stack, yes. If you don't want the stack to stick around, have a Rust wrapper.

- J: Patrick's proposal was to not have split stacks, let the VM system deal with it?

- N: Bill M. from JS team was on the Capriccio team, which was one of the split-stack implementations. I talked to him and he said "why do you care, doesn't Rust target 64-bit?"

- D: I thought mobile would be 32-bit for a while

- N: Probably so

- P: The performance costs of thrashing on a stack boundary are totally unacceptable

- D: Before we make absolute proclamations, let's lay out the various things that are in tension here. This is a big conversation, I don't want to jump to "we must do X because it's the end of the world." I'm not ready to say "this trumps everything" before we talk about what "everything is".

    I'm concerned about 2 things: First, it violates the tension with the rest of how we've been designing Rust that this is a much more heavyweight machinery that's done for you behind your back, that you can't control. It's in tension with low-level systems nature of Rust and predictable perf. That said, the two things I see us getting from it are: (a) the ability to have lots of lightweight tasks (tasks as cheap abstraction boundary), and (b) the ability to write recursive algorithms that operate on potentially large data structures.

    And actually, (c), currently in C++ and Gecko, anything that runs out of stack space is an unsafe crash and we have to manually implement things like red zones to safely exit the browser, which is awful. We have wanted to be able to more granularly kill subcomponents of a program, particularly in the browser, but for other Rust programs too. Like the Erlang philosophy of being able to kill components of a program w/o affecting the whole. These are the 3 things I see automatic stack growth helping us with.

- B: On the issue of safely killing tasks, right now we don't even have a viable strategy for what happens when you run out of stack. It's a process abort; we don't have a clear way to make it terminate safely.

- P: We do, but we have to talk about how to make it work

- B: I disagree with P, I don't know how to do it safely. morestack happens between stack frames when there's no landing pad, so stuff leaks

- N: I agree it's hard to solve, but it seems solvable

- K: What are landing pads for functions?

- P: It's basically a finally block

- K: Where destructors get called?

- P: Yes

- D: I don't know nearly as much about what can/can't be done. I'm laying these out as: any solution you come up with has to take all three things into account. If you decide one of the 3 things can't be solved, you have to understand why

- P: Regarding lots of tasks, you can manually set the stack size for your task. All threading systems have this feature

- ?? But if they're not growable, that's useless

- N: What people normally do is write event-oriented code

- P: Also very limited. You don't know if a bunch of your little threads will simultaneously grow up into a big stack and then OOM you

- N: Is that more true than in other parts of memory allocation?

- D: Let's say you're writing a web server in Erlang, spawning an Erlang process for every connection. What do you do when you get DDOSed or slashdotted? How do you limit the number of processes that you spawn? I wouldn't be surprised if there was a simple, very fast front-end that takes in the requests and decides whether to spawn a new process or say "server too busy"

- J: In Erlang the # of processes is a runtime flag for the VM. You say "I want a million and a half"...

- D: And then you just die if...

- J: Yeah, there's a heartbeat process that runs separately that restarts stuff.

- D: But it would be better if you could say "I'm rejecting this request" and not kill the VM

- N: I don't think it's so hard to track how many connections are in flight. And it's kind of a separate thing

- J: A better example might be when somebody's flooding you. In Erlang the main way to do it is to set a socket option (for having a limited receive buffer)




- D: Patrick, I feel like you've reached a conclusion that I haven't reached yet. Seems to me Erlang is an existence proof that ppl are able to go very far. It's a battle-tested approach to have growable stacks. Now of course Erlang is an extremely different design space-

- P: Does it actually use linked lists of stack segments? Or does it have contiguous stacks and copy?

- D: I don't know. Those are different implementation strategies

- P: The contiguous stack approach may be workable, but it will never work for Rust

- N: Have you looked at the Capriccio project? They did it for C

- P: How did they deal with this thrashing problem?

- N: I don't know. It's research work. [stuff I didn't catch]

- P: We've tried hard to make this path smaller and smaller. If you say "it shouldn't be expensive", I don't believe you anymore

- B: I disagree about the amount of effort we've put in to optimizing the stack switching path.

- K: What do you mean by "contiguous stack won't work for Rust?"

- P: We can't copy stacks to other places in memory, because we have interior pointers into the stack, and no metadata

- F: So we can't even do a mixed strategy, then?

- P: Right

- D: We're being driven by LLVM, and one question is if the picture changes if we change LLVM

- P: No. This is fundamental to the language

- D: So you're absolutely clear in your head that there's only one thing we can do, but I haven't heard answers for my 3 questions yet

- P: So when it comes to lots of lightweight tasks, you just have to set a small stack limit

- D: I don't buy it

- P: The cost of global concurrency is X, the cost of this is like 30X.

- D: I'm not asking you to defend your position, I'm asking you to answer these 3 questions. I think it's very unlikely that people will write programs in the style of "lots of little tasks" when they can only do a top-level loop where they can only call 1 function. N says they'll write event-style instead. Maybe our answer is "we don't solve that" but I'm asking you to think through the conclusion. I don't believe the answer is ppl will write actors that can't use function abstractions

- P: If there are a million functions they all have to use a certain amount of stack, regardless of your allocation strategy for it-

- D: I'm not saying any language makes computers infinite

- F: There's a difference between a tree structure for stacks and a linked structure-

- P: We've never talked about that

- F: But it's a way you could get lighter weight

- P: But we're not talking about that. We're talking about the stacks for each thread being entirely disjoint. Whether they're allocated as a linked list or contiguously doesn't change the fact that a task that needs n bytes of stack needs n bytes of stack

- N: There's optimistic assumptions that you're not- they don't need all the stack all the time. Still need rate limiting and stuff like that...

- P: Let's be clear: We're talking about a situation where most tasks spend most of their time in a small function that doesn't take much stack. Every so often, not very commonly, they'll call out to another function that requires a big stack. We have to assume they won't all do this at the same time. This is the scenario where small stacks-

- N: They can do it, some of them may fail

- P: If they fail to allocate, they're dead

- D: And then you have a task-local failure, which has a well-defined semantics. I'm not taking anything anybody says here lightly. Semantically, at least, that's an understandable situation.

- P: Brian's pointing out that would also be a process abort

- D: I'm also saying that's a problem.

- P: You're misunderstanding. It would be a process abort in either situation

- D: I'm not saying that's not a problem. We have to deal with this thing of we can just abort the process. If the answer is "sorry, we can't do it", then that's the answer. I don't know if this is the right forum for this.

- P: I think the process abort problem is a separate conversation

- N: But it's at least solvable

- P: In theory it should be possible

- N: I'm not saying I disagree w/ you either, but I'm also not at this conclusion. If we don't do it, there's no other solution (in libraries, etc.) other than writing code in event-oriented style

- B: Somebody proposed that we keep the split-stack mechanism but default to big stacks.

- P: strcat had a good argument that this whole conversation is irrelevant to Servo. Servo doesn't need very many small tasks

- D: Hundreds but not ten-thousands?

- J: Probably some constant times the number of tabs

- D: but I thought tabs were separate processes. then each process would get to start from a fresh Rust VM

- P: It's kind of complicated because of different constellations. Because of pop-up windows and inter-tab communications, some will be processes and some will be tabs. Actually, it's origins times CPU cores because there will be thread pools

- N: This is because we're doing event-driven IO

- D: If each one of those tasks gets a 2-MB stack, is that manageable? Does that make Servo bloated?

- P: I think 2 MB is too big

- J: I don't know the answer to that question

- P: 2 MB was added because it made the JIT work in some cases

- B: We shouldn't even have done that

- P: 2 MB is not the right answer. Should be more like 64K. I don't think we'll ever get that low. 

- N: Are you sure? Have you seen what printf does?

- P: We need to fix printf. I mean, honestly, I'd be totally OK with having split stacks and just having the default spawn spawn a big stack. All I want is to get rid of this thrashing issue

- B: If we keep split stacks, but make them big, that doesn't solve our FFI issues. Well, it might because it might mean FFI will just run on the big stack. But still doesn't solve issue of whether we have wrappers, and how the ergonomics around that work. Will still have to annotate who claims that big stack.

- N: It's not clear to me that we should keep split stacks if we don't plan to use them

- J: A worry is that we'll have big stacks and not be able to use them

- P: Niko's idea of having compiler yell at you if you do it wrong would help (in the presence of small stacks). May be slow because you didn't test performance in small-stack scenario

- J: Here's where the big-stack solution could be bad. If you're doing a network server, you might want to call bcrypt() first. Would shell out to a C thing for that. So everything would have a big stack.

- N: Should call a Rust function that calls bcrypt(). Best practice according to us is to have wrappers.
- P: I've justified this to myself by thinking "you usually have to write a wrapper anyway", to adopt coding styles if nothing else. There's also the unsafe issue, whereby if you don't wrap them, you're forcing unsafe on all your callers

- J: So you would wrap bcrypt not with your function, but with some kind of other task -- if 1000 people connect at once, you've just allocated 2000 MB of stack. You get a call to bcrypt for every incoming connection, right?

- B: the wrapper function both acquires and releases a big stack solely for the FFI call. You only do it once per thread.

- N: we only have 8 cores, no pre-empting, it's not all going to run simultaneously

- B: It's how the old FFI works, you acquire the stack only for the duration of the call

- P: Re: Graydon's comment, I understand you and I disagree but I'm not convinced optimization work will make this not a perf. problem. That code is already super-complicated.

- D: I still haven't heard any answers to some of my questions, like recursion. If the answer is "recursion doesn't matter" I'll point you to Spidermonkey, which does write a lot of its algorithms recursively, and just blows up

- P: Re: recursion, if you hit stack growth in your recursive algorithm, your algorithm will be super slow. Slow enough that it... it depends on your use case. It may be people use Python for a lot of things and Python is super slow, so maybe it's fine. In Spidermonkey, slow algorithms are not acceptable. So people wouldn't use recursive algorithms in the split-stack case if they depended on split stacks in performance-critical code. Your algorithm will work but it will become super slow.

- D: That's fair, but if you have small or fixed-size stacks, then you actually just can't use recursion, up to a certain size. It's hard to reason about when you can or can't use it, because it depends on constant factors to determine how large a data structure will terminate your process (or fail your task). You can write a recursive alg. and optimize it later and it'll actually work, with split stacks. You just have performance bottlenecks you can clean up later. But if it crashes your browser, you just can't use it. Remember, Firefox 1.0 was not a fast browser.

- P: A lot of functional languages don't use split stacks...

- D: All FLs assume you have unbounded stack space

- P: That's not true

- D: Sorry, Ocaml. The vast majority...

- P: Anything on the JVM won't assume that

- D: The tradition of FLs is recursion is inherent to programming and your RTS doesn't impose unreasonably small restrictions on how deep recursion can go.

- P: To be clear, that's not the case today in Rust; we have a limit

- B: 10 GB (edit: actually 1GB)

- D: 10 GB doesn't concern me, because that's a reasonable heap limit. I'm not asking for infinite computers. Just asking for us to have a story for programmers about what you have to do in Rust; if the answer is "don't use recursion", that's something we should be clear about if it's your decision. Similarly, with tasks, if it's "don't spawn a whole lot of tasks" we need to be explicit about that. I'm not comfortable with it. I don't have a solution, so I won't block anybody, but I don't hear you articulating the big picture. There's more constraints than just this performance issue of thrashing, and a solution has to address all these constraints

- P: It doesn't have to address all the constraints

- D: It does, even if the answer is "don't use this feature"

- P: I don't know how much farther we can get. We've observed a performance issue, and that's in tension with the issues of your program going slow instead of crashing if you use too much recursion. I'm not sure how much more can be said on that.

- D: So your feeling is it should just crash? I was just asking you to state your opinion; just want to know what the programming model is.

- P: Originally, I was thinking it should crash the task, not the process. I think having the process crash is unreasonable no matter what we pick. We need to have a solution to make the task crash and not the process

- D: I wasn't clear that was your opinion; I'm thrilled to hear you say that

- S: Do we have good numbers about how slow these stack switches are? What's the big cost there?

- P: I have measured FFI processes a lot. For example, stack switching means 90% of layout, roughly, in Servo is stack switching

- S: This is doing the stack switch into-

- P: Into C. Essentially the same path-

- B: Not quite-

- S: It'll probably be hitting it more often, because there's no way to avoid it right now

- P: There is the fast FFI, but it's not on by default. Fast FFI does help a lot, but it only helps when you already have a big stack. (due to the morestack path being slower -- there's just a lot of code there)

- D: My feeling is if we could get it so stack overflow is a task-local failure, that's a pretty coherent design. It says Rust has a thinner, lighter-weight stack mechanism that limits some of the use cases that we thought we'd be able to handle: particularly, recursion and variable-sized tasks. I'd be much happier if there were some way -- even if it's the uncommon case -- to have recourse, as a programmer, to be able to do that without having to defunctionalize your entire recursive algorithm into manual stacks and loops. Even that, systems programmers understand they have to use loops instead of recursion, but it seems like a shame; would be nice to have a fallback. I buy that it wouldn't be the common case.

    I'll just mention this one totally naive ridiculous thing I brought up: if there were some way to have configurable stacks, where you could write unsafe code to follow some protocol; I imagine the hard part is how to specify a protocol that avoids the specific details of a specific compiler. I'm just wondering if there's some minimum subset of stack functionality that could be turned into a trait, like just the allocation policy, and not the argument-passing policy

- P: There's also strcat's proposal, which is to have small stacks but not have them be the default. Then there's Niko's proposal to require annotations

- G: I hate to write here as a form of conversation, but .. patrick: why are malloc and free in our profiles? doesn't that strike you as a bit wrong? (can someone ask?)

- G: try to be absolutely clear about what you're measuring. If your 90% number includes malloc and free, you're measuring something else.

- G: why would morestack be big? 

- D: It seems heavyweight to turn it into an effect

- P: It's just the effect that the FFI burdens you with -- it's already unsafe

- D: I think the thing that's missing from that is it doesn't provide you with stack growth. It's only big or small

- P: No, strcat's proposal is that we keep stack growth-

- D: But that means by default we're taking a big performance hit. With strcat's proposal, the default is growable stacks-

- B: The initial stack segment would be huge, so you would almost never grow the stack

- D: That seems to me like a situation where you could have some sort of lightweight, configurable approach based on traits where you could implement a policy that says "I don't ever want growth". Or maybe you just need a boolean flag. It would be good to have the functionality to be able to grow tasks, but allow programmers to opt out of it. strcat's direction sounds plausible to me, but we can continue this conversation elsewhere.