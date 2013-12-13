## On-the-fly Syntax Checking
Install `flycheck` from the Package Manager.

Use `flycheck-mode` in your Rust buffer and it will highlight syntax errors on the fly. It uses the default `rustc` on your machine. 

To make it use a particular rustc binary, you have to create your own syntax-checker. Like this one, for Servo.
``` lisp
(flycheck-define-checker servo-rust
  "A Rust syntax checker using the Rust compiler in Servo."
  :command ("/path/to/servo/rustc"
            "--parse-only"
            source)
  :error-patterns
  ((error line-start (file-name) ":" line ":" column ": "
          (one-or-more digit) ":" (one-or-more digit) " error: "
          (message) line-end))
  :modes rust-mode)

(add-hook 'rust-mode-hook (lambda () (flycheck-select-checker 'servo-rust)))

```

## Simple Auto-Completion
For basic auto-completion of symbols available in open buffers, use `M-/` (`dabbrev-expand`)

## Auto-Complete mode
This lets you view the potential completions in a pop-up right after point. You can choose among the candidates using `M-n / M-p`

## Imenu: Finding definitions within a file
Use `M-x imenu` to choose from a list of all the items in a file (e.g., functions, structs, traits, etc.).

### Idomenu
Install the package `idomenu` from the Package Manager.

Now `M-x idomenu` will let you choose from the items in a file, using Ido completion (fuzzy completion).

## Tags

### To create the Tags table for Emacs
Assuming RUST_ROOT is where the Rust source is, and PROJECT_DIR is your project
``` bash
    RUST_ROOT=~/rust
    RUST_OPTIONS_FILE=$RUST_ROOT/src/compiler/rust/src/etc/ctags.rust
    PROJECT_DIR=~/project
    ctags-exuberant -e -f TAGS.emacs --options=$RUST_OPTIONS_FILE -R $PROJECT_DIR
```
will build the tags table in TAGS.emacs (may take a while if it is a big project like Servo or Rust itself)

You can then jump to the definition of any function or struct or enum by using `M-.` (`Alt + .`)

## Moving around the buffer
Emacs motions like `C-M-a` and `C-M-e` (`beginning-of-defun` and `end-of-defun`) work out of the box.

So does `C-M-h` (`mark-defun`).

You can use them to manipulate Rust items - functions, structs, enums, etc.
