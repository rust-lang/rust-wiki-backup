When you run `make install` or install rust from a package, you will get a tool called `cargo` as well.

Cargo is a decentralized system for registering, finding, installing, updating and otherwise "managing" collections of rust source code, libraries, and similar "packages". The definitions here are all somewhat loose since the conventions of use for cargo are still evolving; we encourage new rust users to register any rust code they publish by name with [cargo central](http://github.com/mozilla/cargo-central), our central cargo package list, so that other users can find your work and easily install it.

## How it works

Cargo keeps a list of "sources" from which it loads "package lists". Sources are described in a "sources file" kept in `$HOME/.cargo/sources.json` file, a plain-text JSON file listing named URLs, and package lists are, themselves, JSON files loaded from the URLs specified in the sources file.

* `cargo help` will give you a list of commands.
* `cargo init` will set up a `$HOME/.cargo` directory in which packages and package-lists will reside.
* `cargo sync` will fetch package lists from the "sources" configured into the `$HOME/.cargo` directory.
* `cargo list` will list packages registered with existing sources
* `cargo install <package>` will install a package by name into the `$HOME/.cargo/lib` directory

The `$HOME/.cargo/lib` directory is part of the default library search path used by `rustc`, so you should be able to `use` a library crate directly after successfully installing with cargo.

## Registering your own packages

We encourage all users to register their packages with [cargo central](http://github.com/mozilla/cargo-central), the source that is fetched by default, by any new cargo user. This will give the widest visibility to your package and make it easiest for other developers to find and fix packages that might have fallen behind changes to the language, upstream.

To "register" a package, the easiest approach is the following:

  * Push your package to a git repository somewhere. Make sure your package has a crate file in its root directory called `<yourpackage>.rc`, that contains a suitably detailed `#[link ...]` attribute describing at least its short name. We will likely make some link attribute fields mandatory for future inclusion in cargo -- possibly the `uuid` field or `vers` for version numbers -- so it's good to include those as well.
  * fork the `cargo-central` repository and add a new entry to the `packages.json` file, for example:
        {
        "name": "crypto",
        "uuid": "38297409-b4c2-4499-8131-a99a7e44dad3",
        "url": "git://github.com/elly/rustcrypto",
        "method": "git",
        "tags": ["crypto"]
        },
  * submit a pull request and, if you want it expedited, notify someone on IRC or through email who has write access to the cargo-central repository. 

Once you've registered your package like this *once* you no longer have to interact with `cargo-central`, cargo has been directed to use `git` to access your repository. So as you push updates to that repository, on your own time, they will immediately become visible to users performing `cargo install <yourpackage>`.

That's it! We've tried to keep it as simple as possible. The system is structured in a decentralized fashion -- you can add your own sources, you don't *have* to use `cargo-central` -- but we encourage you to use `cargo-central` to help us keep track of all the packages being written, and try to keep name collisions to a minimum.