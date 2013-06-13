Rust crate versioning: "very explicitly designed to meet the needs of linux packagers" - pcwalton

### Key points:
* Current behavior is not what's planned+accepted as design.
* Crates have hashes and versions--both work together to provide versioning and safety.
* Crate Hashes are not versions.
* Symbols also have hashes and versions, with somewhat different semantics.

### Crate filenames and hashes:
* Library filenames look like: `lib<shortname>-<hash>-<version>.so` (or possibly `lib<shortname>-<hash>.so.<version>` in the future)
    * Example: `librust-39583f72884834e3-0.6.so`
    * The version number is not included in `<hash>`
* `<hash>` is intended to prevent different authors' libs with the same `<shortname>` from conflicting.
    * Two authors decide they want to make `rustsdl`&mdash;no problem, as long as they pick different hash inputs.
        * Hash inputs:
            * Currently the link meta tags in the crate: `#[link(uuid="..", url="..", etc.)]`;
            * Planned to hash only the pkgid: `github.com/yourname/packagename`, `mypkg.freedesktop.org`, etc.
        * Users should plan for the transition from "bag of meta tags" to "single per-crate pkgid".
    * Hashes should be stable, based on identity and not time-variant state of the code.

### Symbol hashes:
Each symbol in a crate is also versioned. Symbols have C++-mangled names of the format: `<crate-local-name>::<hash>::<version>`
* Example: `cleanup::annihilate::_c0b9e988aa1ab6bf::_07pre`
* Again, `<hash>` and `<version>` are orthogonal:
    * `<hash>` is computed based on the *crate* hash and type of the symbol in the Rust type system.

This setup is necessary in order for symbol resolution to make the same name-collision safety guarantees that crate filenames have. It's also used to simulate per-crate symbol namespaces to mirror Rust semantics at the linking stage. Linux uses a flat symbol table, so without `<hash>` and `<version>`, symbols from disparate crates or versions of the same crate would collide in the PLT.

Meanwhile, the type input to the hash ensures that types of functions cannot change without causing runtime linking failure for dynamically-linked binaries--unlike C, you cannot change the type of a symbol without recompilation, because that would effectively transmute arguments and return values.