# What's Rustpkg?

Rustpkg is a tool that brings features such as a build system described in build scripts.
It's a purely functional package manager that has no central sources of any kind,
but rather installs via URLs. It's similar to how Go's `go get` tool works, except
Rustpkg requires a central metadata file (`pkg.rs`) in the repository, archive or folder
in order to figure out how to build the package. This is a side effect of Rustpkg
allowing multiple crates to be defined in a single package (i.e. a package is defined
as a set of crates rather than a package being exactly the same as one crate).
There's a plan to make it so the `pkg.rs` is not needed for single-crate packages,
if you're not a fan of having to create a `pkg.rs` file and would rather it
just find `.rc` files.

The metadata is written in Rust. This is both good and bad. One con is that
Rust's syntax is going to be moderately unstable until v1.0 (as I was told),
so compiling an old package might spring up weird compiling errors. On the other hand,
you get to write the build process in Rust itself which is so incredibly meta and fun
and you get to use an awesome language to build projects in that same awesome language.

Rustpkg also doubles as a powerful build system that gives you two ways to
describe the build process of projects: declarative and imperative. The declarative
syntax allows you to declare the build process using Rust's attribute syntax.
A simplistic declarative package script (`pkg.rs`) would be something
like the following (Servo doesn't actually use Rustpkg, it's just an example):

```rust
#[pkg(id = "org.mozilla.servo",
      vers = "0.5.6")];

#[pkg_dep(url = "git://github.com/mozilla-servo/rust-xlib")];

#[pkg_crate(file = "src/servo.rc")];
#[pkg_crate(file = "src/servo-gfx.rc")];
```

The imperative API is for when you need a more powerful build process, such
as probing the system for important configuration, running shell scripts (or
even autotools, which is a planned built-in feature that I want to add). A simple
example using the imperative API is as follows (Rustpkg is automatically linked into package scripts):

```rust
#[pkg(id = "org.mozilla.servo",
      vers = "0.5.6")];

#[pkg_dep(url = "git://github.com/mozilla-servo/rust-xlib")];

#[pkg_do(build)]
fn build() {
   let platform = if os::is_toaster() { ~"platform=toaster" }
                  else { ~"platform=alien" };
   let crate = rustpkg::Crate(~"src/servo.rc").cfg(platform).flag(~"-g");

   rustpkg::build(~[crate]);
}
```

When a package's crates are installed, they are stored under a unique name (Rust's library crates
work like this out of the box) in `~/.rustpkg/bin` or `~/.rustpkg/lib`. This allows
packages to coexist in a purely functional manner. However, if you want to easily
use a binary crate that has been installed from the `~/.rustpkg/bin` path (after
adding it to your `$PATH`) then rustpkg provides "preferring" functionality
to symlink a generic name for the crate into the binary directory (which
voids the purely functional label, which is why it's explicitly optional).
For example, if you had a crate with the name `servo` and it's uniquely installed to
`~/.rustpkg/bin/servo-<hash>-0.5.6` then `rustpkg prefer servo` would link it to
`~/.rustpkg/bin/servo` which you can then easily run as just `servo` if the binary
directory is in your `$PATH`.

All crates are built and installed into `~/.rustpkg`,
but I'm planning on changing it so that crates are initially built into `.rustpkg`
without unique hashing (only for binaries.. obviously library crates will still
need the hashing kept intact). This allows you to easily run a built crate without installing
it (I'll probably provide a `rustpkg run <crate>` command to compile and run a crate).
Crates are then copied from `.rustpkg` to `~/.rustpkg`. This patch will also remove uninstalling
packages because it's not really needed for now.

Preferring is currently pretty dodgy, along with some of the other features. But that's to
be expected due to this being the initial implementation and I plan to perfect it
over time (including documentation). I still really want to add the following
but they're not important for the initial pull request:

* More features of the imperative API
* Use the JIT for package scripts (JIT segfaults for me at the moment)
* Use `std::workcache` for only compiling things that have changed
* Add wrappers around autotools, `$CC`, etc. that also use workcache
* Allow custom configuration to be declared for dependencies (`#[pkg_dep(url = "git://github.com/brson/rust-sdl", target = "image mixer")];`)

If you need any more help, either give me a ping on #rust or check out `rustpkg --help`.