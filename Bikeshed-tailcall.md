This page is for collecting ideas around axing or implementing tail calls in rust. The current status of this issue is undecided after a longer discussion on the list in jul/aug 2001. (There currently is no consensus on the matter nor is this page a complete summary of that thread, I am just writing down a few own thoughts right now but the page may very well become that -- boggle)

## Pro/Con

* Pro: Obvious uses in functional programming (But: Replacable with iterators/blocks in many cases)
* Pro: People expect that in a language that says to support functional programming with immutability as a default (But: Well     
  that really just is a sales argument and should not steer design)
* Pro: Simple, forwarding functions don't pay the cost of an extra stack frame
  * this is currently used in core::float to pass calls on to core::f32 and core::f64 depending on the target    
    architecture (even though "be" is not really implemented)
  * it may be very relevant for supporting delegation in the object system or typeclass proposal
  * but: is delegation/forwarding really that important
  * but: maybe can be supported with a different mechanism
* Pro: The classic way to implement actors is by having them loop on a tail callable function
  * The interesting bit about this is that it becomes easier to change actor behavior at runtime which may help
    with code migration (atomic switch point) in some very distant future (erlang does it that way) and supports
    implementing complex control flows over multiple actors by sending closures (did a paper on that, nice if actors
    correspond to "stages")
  * But: ORLY, second guessing the future? 
  * But: Other solutions possible
* Con: Tail calls cannot be implemented in general without a serious performance hit for all calls (But: There may be sensible, limited forms of tail call support that avoids this)
* Con: Rustic code isn't that likely to need tail calls (But: until you code an algorithm that cries for it)

## Make it explicit

Goal: (1) Tail calls where possible, mostly for in-crate use, as an implementation technique (2) but allow cross-crate passing of tail callables for the few situations where this might be desirables (actors)

Ok, the main problem seems to be the overhead of making every function tail callable and having to respect that at call sites, too. Thus I suggest to make "tail callability" an explicit part of function types (like "pure"), i.e.

    tail fn foo(x: int) -> int { x + 1 }

    fn bar(y: int) -> int { be foo(y+1); }

where the type of `foo` would be `tail fn(int) -> int`.  Additionally, at least within a crate, there could be a

    bind tail fn bar(_)

expression for creating tail-callable wrapper functions of normal functions in scope. Inversly, tail callable functions could be bound into non tail callables using regular `bind`. Alternatively, leave the generation of wrappers to rust and
signal an error where they cannot be made.

Arguably partitioning function types into tail and non tail callables is an implementations aspect that just should not surface on the type level (at least in interfaces) as it hurts homogeniety as well as substitutability. Yet sensible uses most likely are limited, crate-local, and related to implementation where total abstraction (i.e. being able to call with any function irrespective of it's tail callability status) is not as important. 

The exception will be in "strategy pattern instances", like abstract actor main loop functions. However, here it seems rational to require tail callability as part of the interface since it is part of the contract of instantiating the strategy.


Question: why does implementing tail calls necessitate slower function calls? Is this because the compilation strategy currently used would necessitate a trampoline approach?
