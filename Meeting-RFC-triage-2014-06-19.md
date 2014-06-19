# RFC triage 2014-06-19

# Attending

huon, acrichto, aturon, huon, niko, nrc, luqman, zwarich, brson, cmr

# Action Items

- (brson) close https://github.com/rust-lang/rfcs/pull/39
- (brson) close https://github.com/rust-lang/rfcs/pull/76
- (brson) close https://github.com/rust-lang/rfcs/pull/78

# PR 39 - allocator trait

- nrc: want to do it, but probably needs new RFC
- brson: why is this not good enough?
- cmr: doesn't reflect current thinking
- niko: tied to GC design. not ready to accept this

# PR 76 - multiple trait impls for single impl block

- nrc: recommend postpone
- all: agree

# PR 78 - extend safe mutability

- nrc: from mutpocalypse discussion. thought we had agreed to close this previously
- zwarich: brings back aliased mutable data. big change
# PR 81 - tail call elim

- cmr: already closed
