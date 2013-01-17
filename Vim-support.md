Copy the content of the Rust repository's `src/etc/vim` directory to `~/.vim`.

As long as `filetype plugin indent on` is used in `~/.vimrc`, .rs files will be
detected as Rust and the correct indentation and syntax highlighting will be
used.

By default, some operators like `->` are concealed with the unicode
representation. This can be disabled by setting `let g:no_rust_conceal = 1`.
