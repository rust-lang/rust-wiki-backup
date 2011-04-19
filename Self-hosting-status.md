# Self-hosting status

## Typechecking

All modules typecheck.

## Translation

* ☑ `front.ast`
* ☑ `front.creader`
* ☑ `front.extfmt`
* ☑ `front.codemap`
* ☑ `front.lexer`
* ☑ `front.parser`
* ☑ `front.token`
* ☑ `front.eval`
* ☑ `middle.metadata`
* ☑ `middle.fold`
* ☑ `middle.resolve`
* ☑ `middle.capture`
* ☒ `middle.trans` &mdash;infinite loop on line 1024 (`if (cx.kind != NON_SCOPE_BLOCK)`. `trans_compare_inner`/`trans_compare`/`iter_structural_ty_full` infinite loop.)
* ☒ `middle.ty` &mdash;bad LLVM cast while translating ty_to_str.fn_to_str
* ☑ `middle.typeck`
* ☑ `middle.typestate_check`
* ☑ `back.abi`
* ☑ `back.x86`
* ☑ `driver.rustc`
* ☑ `driver.session`
* ☑ `pretty.pp`
* ☑ `pretty.pprust`
* ☑ `util.common`
* ☑ `util.typestate_ann`

Feel free to add to this list; just move modules around in `rustc.rc` to see what works and what doesn't.
