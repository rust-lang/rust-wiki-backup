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
* Con: Tail calls cannot be implemented in general without a serious performance hit for all calls (But: There may be sensible, limited forms of tail call support that avoids this)
* Con: Rustic code isn't that likely to need tail calls (But: until you code an algorithm that cries for it)

## Make it explicit

Ok, the main problem seems to be the overhead of making every function tail callable and having to respect that at call sites, too. Thus I suggest to make "tail callability" an explicit part of function types (like "pure"), i.e.

    tail fn foo(x: int) -> int { x + 1 }

    fn bar(y: int) -> int { be foo(y+1); }

Additionally, at least within a crate, there could be a

    bind tail fn bar(_)

expression for creating tail-callable wrapper functions of normal functions in scope. Inversly, tail callable functions could be bound into non tail callables using a normal `bind` call.