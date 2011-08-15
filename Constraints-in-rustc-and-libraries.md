This page is an attempt to catalogue places in the compiler, and in the standard Rust libraries, where the addition of a typestate constraint would improve quality.

_This list is incomplete. You can help by adding to it._

## Compiler

### Front-end

#### Types

`ast::path_` -- could give this type a constraint indicating that `idents` and `types` have the same length and are non-empty. Requires a call to `ivec::len`, which in turn requires language changes (see [[Proposal for predicate language]]). Could be implemented without language changes by reimplementing `ivec::len` as a pure, recursive predicate.

`node_id` -- could have constrained types that relate a `node_id` with a context, inducing a type-based distinction between node IDs for `uses` and node IDs for `definitions`.

### Middle-end

#### Types

`tstate::tritv::t` -- could have a constraint saying that `uncertain` and `val` have the same length, and that for all indices `i`, it's not the case that `uncertain.(i)` and `val.(i)` are both 1. (The latter part seems hard.)

#### Functions

`trans::GEP_tag` -- could give this function a precondition expressing that the argument `ix` must be ≥ 0 and ≤ the number of variants that the argument `tag_id` has. Specifying the latter constraint involves looking up some information in the type context (which is also an argument), but it should be do-able as a pure predicate.

`trans::trans_be` -- could make the assertion `is_call_expr(e)` a precondition instead. (Is `is_call_expr` a pred?)

* It's a pred now -- but this points to the usefulness of syntax for checking that "`e` is built with tag `expr_call`" without having to write out a pred to do it.

## Libraries

### std::ivec

#### Functions

`zip` -- could have a precondition indicating that both arguments have the same length. Same caveats as for `ast::path_`.