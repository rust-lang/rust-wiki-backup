Apr 6 2011: MoCo meeting. Graydon, Patrick, Dave, Marijn talked near-term semantic changes (post-bootstrap). Notes from this meeting.

Goal was to simplify and orthogonalize concepts in types, effects and layers; capture additions we want to make (lambdas, non-copyables), remove things that seem to be either expensive (implicit deep copies, gratuitous boxing or refcounting) or cognitively burdensome (gc kind, effect system), improve programmer control and efficiency.

Conclusions:

 - Nomenclature: 'layer' becomes 'kind', it's a more accurate term.
 - Add uniquely-owned box type ~. Referent and reference lifetimes are 1:1. Can move, swap, deep-copy only.
 - Add 'tree' kind: types with no @ nodes in it, only ~
 - Add 'uniq' kind: types that can't even be copied (see below)
 - Add 'imm' kind: types with no mutable
 - Kind system is therefore 2-dimensional: shared/tree/uniq and stateful/imm.
 - Remove effect system; it's doing more harm than good.
 -- Unsafe effect will turn into unsafe blocks, a la C#
 -- Impure effect will be subsumed by re-splitting from fn into pred/fn, and a few changes to kind system.
 -- preds have fn type when used as first class, can only call other preds, cannot do any escaping mutation.
 -- kind bounds change from raising to lowering bound: plain "T" means shared mutable; "tree T" and "uniq T" are one dimension of kind-bounding, "imm" is second dimension: "fn foo[imm tree T]" or "fn foo[imm uniq T]" also possible. But that's as bad as it gets. NB: this bounding form may well not work, as it's not a simple kind lattice. 
 - Move destructors from objects to independent "resource" uniq type declared with "res(ty t) { .. drop ... }".
 - fn and obj are uniq types: can't copy at all (neither through channels or otherwise)
 - 'lambda form' (probably just "fn(...){...}" in expr context) creates a local fn value that captures environment by alias; can be passed by alias but can't be moved, can't escape to heap since uniq.
 - @fn and ~fn expressions can be used to make a boxed fn; they can only capture shared @vars, nothing by alias. Possibly, if we have a fancy enough analysis, they can copy a non-escaping, never-mutated interior local as well (marijn wants this).
 - Remove 'gc' kind: it's no longer relevant. Resources can only wrap tree or imm nodes, neither of which are cyclic. 'gc' only mattered for control of dtors, which is subsumed by this rule.
 - communication cost model is obvious and predictable given kind reasoning: tree will be constant cost crossing coroutine and thread barriers, costly over processes. imm will be constant cost only crossing coroutine barriers, costly over thread and process barriers. As you'd expect.
 - channels can only send stuff that's shared-immutable or tree. Not shared-stateful, not uniq.
