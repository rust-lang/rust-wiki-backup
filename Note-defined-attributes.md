This page catalogs the item attributes known to the compiler or used by convention.
See [Rust Reference Manual](http://static.rust-lang.org/doc/master/rust.html#attributes) for documentation of frequently used attributes.

Crate
-----

* link
 * name
 * vers
 * uuid
 * url
* comment
* license
* copyright
* crate_type
* no_std - Don't link libstd automatically
* no_uv
* feature - Enable experimental/unstable features. See `src/librustc/front/feature_gate.rs` for feature list.
* no_main

Item
----

* doc
 * doc(hidden)
* allow, deny, forbid, warn - Set lint options
* cfg - `#[cfg(unix)]`, `#[cfg(target_os = "linux")]`, `#[cfg(stage0)]`, `#[cfg(test)]`, ...
* deprecated, experimental, unstable, stable, locked, frozen - item stability
* export_name, no_mangle - Set exported name of item
* link_section
* address_insignificant
* crate_map
* static_assert
* no_freeze, no_send
* unsafe_no_drop_flag
* packed
* simd
* repr
* unsafe_destructor

Mod
---

* path
* link_name
* link_args
* nolink - Don't perform the default linking for a native module
* macro_escape

Function
--------

* test
* bench
* should_fail - The test case should fail.
* ignore - Ignore test on some configuration e.g. (`#[ignore(cfg(windows))]`) or on any case (`#[ignore]`)
 * reason - `#[ignore(reason = "not implemented yet")]`
* inline
 * always, never
* lang
* fixed_stack_segment
* main
* start - Program entry point
* no_split_stack
* cold
* rust_stack

Obsolete
--------

* lint
* abi - use `extern "ABI" fn` syntax
* auto_encode, auto_decode - use `#[deriving(Encodable)]` and `#[deriving(Decodable)]`

???
---

* missing_doc
* no_implicit_prelude