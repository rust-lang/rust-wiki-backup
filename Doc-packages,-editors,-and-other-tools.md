## Packages

* FreeBSD
  * [FreeBSD Port](http://www.freebsd.org/cgi/cvsweb.cgi/ports/lang/rust/) - maintained by jyyou
* Linux
  * [Arch Linux Official](https://www.archlinux.org/packages/community/x86_64/rust/)
  * [Arch Linux Nightly Build Repository](http://pkgbuild.com/~thestinger/repo/)
  * [Fedora Copr](http://copr-fe.cloud.fedoraproject.org/coprs/fabiand/rust-unofficial/) - ([instructions](http://dummdida.blogspot.de/2013/05/mozillas-rust-in-fedoras-ppa-copr.html)) maintained by fabiand
  * [Ubuntu PPA](https://launchpad.net/~hansjorg/+archive/rust)
  * [Gentoo, overlay rust](https://github.com/Heather/gentoo-rust)
* Mac
  * [Fink](http://fink.cvs.sourceforge.net/viewvc/fink/dists/10.7/stable/main/finkinfo/languages/rust.info?view=log) Also available through fink's apt at `deb http://brendan.users.finkproject.org/10.8 stable main` - maintained by bcully
  * [Homebrew](https://github.com/mxcl/homebrew/blob/master/Library/Formula/rust.rb)
  * [Homebrew Cask](https://github.com/caskroom/homebrew-cask/blob/master/Casks/rust.rb)
* NetBSD
  * [pkgsrc-wip](http://pkgsrc-wip.cvs.sourceforge.net/viewvc/pkgsrc-wip/wip/rust/)
* Windows
  * [NuGet package for nightly builds](https://www.nuget.org/packages/Rust/)
    * [[How to install an unofficial nightly for Windows|Doc how to install an unofficial nightly for Windows]]

## Editors

* BBEdit
  * [Erik Rose's BBEdit plugin](https://github.com/erikrose/rust-bbedit-plugin) - Plugin for BBEdit. Probably quite out of date.
* Eclipse
  * [ianb's Eclipse plugin](https://github.com/ianbollinger/oxide) - Eclipse! (dead)
  * [Reidar Sollid's Eclipse plugin](https://github.com/reidarsollid/RustyCage) - A newer Eclipse plugin.
* Emacs
  * [rust-mode](https://github.com/rust-lang/rust-mode). 
  * Flycheck has support for on-the-fly syntax error detection. Use the Package Manager (ELPA) to install `flycheck-mode`
* Geany
  * Since 1.24. [Nightly builds](http://nightly.geany.org/) or the [git master](https://github.com/geany/geany) may have additional improvements.
* Gedit
  * included, see `src/etc/`
* Intellij
  * [Vektah's Intellij plugin] (http://plugins.jetbrains.com/plugin/7438) - Can be installed from Plugins settings in app
* Kate
  * [Rust Kate Config](https://github.com/rust-lang/kate-config) (the file can be copied or linked to ~/.kde/share/apps/katepart/syntax/ for KDE4 or ~/.local/share/katepart5/syntax/ for KF5)
* Netbeans
  * [Rust NetBeans 8+ Plugin](https://github.com/azazar/rust-netbeans) - Forked drrb's NetBeans Plugin
  * [drrb's NetBeans Plugin](https://github.com/drrb/rust-netbeans) - Rust NetBeans Plugin
* SublimeText
  * [kib2's SublimeText2 language file](http://kib2.free.fr/Falcon/blog/25-01-2012-SublimeText2-Rust.html) - Highlighting for SublimeText 2
  * [dbp's SublimeText2 (now maintained by jhasse)](https://github.com/jhasse/sublime-rust) - Probably more up to date than the above.
  * [SublimeLinter plugin](https://github.com/oschwald/SublimeLinter-contrib-rustc)
* TextMate
  * [tomgrohl's textmate bundle](https://github.com/tomgrohl/Rust.tmbundle) - TextMate 1
  * [elia's textmate bundle](https://github.com/elia/rust.tmbundle) - TextMate 2
* Textadept
  * [abaez's Textadept module/lexer](https://bitbucket.org/a_baez/ta-rust)
  * [suhr/ta-rust](https://github.com/suhr/ta-rust)
* Vim
  * [rust.vim plugin](https://github.com/rust-lang/rust.vim). Syntastic has syntax error matching based on rustc.
* NEdit
  * [NEdit syntax highlighting for Rust](https://mail.mozilla.org/pipermail/rust-dev/2013-September/005822.html)
* Notepad++
  * [Notepad++ syntax highlighting](https://github.com/pfalabella/Rust-notepadplusplus)
* Brackets
  * [Brackets syntax highlighting for Rust](https://github.com/frozzare/brackets-rust)
* Atom
  * [Atom Racer : intelligent code completion for Rust based on Racer](https://github.com/edubkendo/atom-racer)
  * [Atom Rust: Syntax highlighting, snippets, etc](https://github.com/zargony/atom-language-rust)

## And more!

* [cargo-lite](https://github.com/cmr/cargo-lite) - A simple package manager
* [Prism's pastebin](http://kib2.free.fr/pastebin) - A pastebin instance that support Rust syntax highlighting using the Prism library
* [startling's pygments plugin](https://github.com/startling/pygments-rust) - Pygments syntax highlighting
* [GitHub linguist](https://github.com/github/linguist) - GitHub's language detection supports Rust
* [lkuper's PLT Redex model](https://github.com/lkuper/rust-redex) - An (out-of-date) model of Rust in PLT Redex
* [highlight.js](http://softwaremaniacs.org/soft/highlight/en/) - Highlight Rust code in your blog by adding a single line of JavaScript to your pages
* [bstrie's map of Rust contributors](http://seleniac.org/map/)
* ["Is Rust fast yet?"](http://huonw.github.io/isrustfastyet/) - graphs of the build/test times of each merge into incoming