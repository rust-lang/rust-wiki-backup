When you run `make install` or install rust from a package, you will get a tool called `cargo` as well.

Cargo is a decentralized system for registering, finding, installing, updating and otherwise "managing" collections of rust source code, libraries, and similar "packages". The definitions here are all somewhat loose since the conventions of use for cargo are still evolving; we encourage new rust users to register any rust code they publish by name with [cargo central](http://github.com/mozilla/cargo-central), our central cargo package list, so that other users can find your work and easily install it.

Although Cargo has the basic functionalities of a package manager, it is very young and constantly evolving, just like Rust. It's still missing a few features that you'll be familiar with from other package managers. For example, at the moment Cargo is rolling-release, which is not intended, the version system is just not yet implemented. Cargo also automatically solves dependencies from crate files, however cyclic dependencies are not yet handled. Cargo may also be potentially removed in the future for a single, united Rust tool that both builds and installs packages/dependencies.

## How it works

Cargo keeps a list of "sources" from which it loads "package lists". Sources are described in a "sources file" kept in `$HOME/.cargo/sources.json`, a JSON file listing named URLs for package lists. Package lists are, themselves, JSON files called `packages.json`, loaded from the URLs specified in a sources file.

The `$HOME/.cargo/lib`, `./.cargo/lib` and `$INSTALL_DIR/cargo/lib` (e.g. `/usr/local/lib/cargo/lib` on Unix systems) directories are in the default library search path used by `rustc`, so you should be able to `use` a library crate directly after successfully installing with cargo locally, globally or to the user's home.

## Commands

All Cargo commands are run using `cargo <cmd>`, where `cmd` is a specific operation/command you want to use. Not passing any command to Cargo will display a list of commands and general help information.

### init

The init command will initialize Cargo in the user's home directory (i.e. `$HOME/.cargo`). Cargo automatically initializes itself. This command will download the official source manifest from the Rust website and add them to cargo. *All previously added custom sources will be lost.*

#### Example

```sh
$ cargo init
info: initialized .cargo in /home/phosphorus/.cargo
```

### install [options..] [args..]

The install command accepts a variety of installation methods. By default, Cargo will install locally in the current working directory (`./.cargo/lib`). You can provide the `-g` option to install to the user's home (`$HOME/.cargo/lib`) and the `-G` option to install to the system directory (`/usr/local/lib/cargo/lib`). You can also provide the `--test` option to [run a crate's tests](https://github.com/mozilla/rust/wiki/Note-unit-testing) before installing.

#### install \[source/]\<name | uuid\>

Install a package by `uuid` or `name`. You may also provide a specific `source`.

#### install \<git url\> \[ref]

Install a package via Git from `git url`. You can also optionally provide `ref`, which will be checked out before installing.

#### install \<tarball url\>

Install a package via curl and a tarball. 

#### install \<tarball path\>

Install a package from a locally-available tarball.

#### install

Install a package from the current working directory.

#### Example

```sh
$ cargo install # installs from the cwd
$ cargo install sdl
$ cargo install central/sdl
$ cargo install git://github.com/brson/rust-sdl
$ cargo install http://example.com/pkg.tar.gz
```

### list [sources..]

The list command will list all sources and their packages, where packages are listed lexicographically. If you provide a list of source names, it  will only list packages from those sources.

#### Example

```sh
$ cargo list
info: syncing source: central...
info: synced source: central
info: central (https://raw.github.com/mozilla/cargo-central/master/packages.json)
   >> bloomfilter, cairo, crypto, csv, elasticsearch, glfw, llvm, mecab, mongrel2, 
   >> mre, mustache, pcre, rparse, rust-hello, rustgl_4_2_core, rustic, rustray, 
   >> rustx, sdl, socket, sqlite, time, tnetstring, uri, uuid, zlib, zmq
```

### search <query | '*'> [tags..]

The search command will search through every source for a specific package with credentials you are looking for.

#### Example

```sh
$ cargo search gl
info: syncing source: central...
info: synced source: central
info: central/glfw (a1848f17-c60e-469d-8b12-8f1b45d2c31f) [opengl, graphics]
   >> glfw bindings for Rust.

info: central/rustgl_4_2_core (72589cd3-f56b-40c3-b944-db257ed8b36c) [opengl, graphics]
   >> Opengl 4.2 bindings for Rust.

info: found 2 packages
```

### sources

List your personal sources.

#### Example

```sh
$ cargo sources
info: central (https://raw.github.com/mozilla/cargo-central/master/packages.json) via curl
```

### sources add \<name\> \<url\>

Add a source with `name` located at `url`. Cargo will figure out how to fetch the source from the URL.

#### Example

```sh
$ cargo sources add central2 git://github.com/mozilla/cargo-central
info: added source: central2
```

### sources remove \<name\>

Remove a source by `name`.

#### Example

```sh
$ cargo sources remove central2
info: removed source: central2
```

### sources rename \<name\> \<new\>

Rename a source from `name` to `new`.

#### Example

```sh
$ cargo sources rename central2 central3
info: renamed source: central2 to central3
```

### sources set-url \<name\> \<url\>

Set a source by `name`'s url to `url`. It will re-calculate the method to use to fetch the source.

#### Example

```sh
$ cargo sources set-url central git://github.com/mozilla/cargo-central
info: changed source url: 'https://raw.github.com/mozilla/cargo-central/master/packages.json' to 'git://github.com/mozilla/cargo-central'
```

## Registering your own packages

We encourage all users to register their packages with [cargo central](http://github.com/mozilla/cargo-central), the source that is fetched by default, by any new cargo user. This will give the widest visibility to your package and make it easiest for other developers to find and fix packages that might have fallen behind changes to the language, upstream. **N.B. Your package entry's `name` field may only contain alpha-numeric characters, dashes or underscores.**

To "register" a package, the easiest approach is the following:

  * Push your package to a git repository somewhere. Make sure your package contains a crate file somewhere in its directory structure called `<yourpackage>.rc`, that contains a suitably detailed `#[link ...]` attribute describing at least its short name. We will likely make some link attribute fields mandatory for future inclusion in cargo -- possibly the `uuid` field or `vers` for version numbers -- so it's good to include those as well. If your package is a library, you will need to add `#[crate_type = "lib"];` to the `<yourpackage>.rc` file.
  * fork the `cargo-central` repository and add a new entry to the `packages.json` file, for example (including a uuid from the `uuidgen` tool):

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