one word value, low-level / fast calling convention:
--------------------------------------------------------

fn(int, int) -> int // a C function pointer. you can copy it, send, etc.
                       // no capture. items and 1st-class copies of items
                       // have this type.


two word values, environment-passing calling convention:
--------------------------------------------------------

fn@(int, int) -> int // shared env-capture

fn~(int, int) -> int // unique env-capture

fn&(int, int) -> int // alias env-capture



purely as a syntactic convenience:
----------------------------------

block(int, int) -> int <===> fn&(int, int) -> loopctl<int>

these two types are equal, though we remember which you typed in. they just
unify. and 'for' loops are built into the language.

Passing &block(int, int) -> int is equivalent to passing &fn&(int, int) ->
loopctl<int> but nobody is seriously going to write that.



as an API-level convenience:
----------------------------

fn~ and fn@ can convert to fn&, but not vice-versa. so you write most APIs to
take &block(...), meaning &fn&(...)



named locals w/ capture:
------------------------

fn& foo(...) { ... }
fn~ foo(...) { ... }
fn@ foo(...) { ... }

are all legal forms of declaring a local, named, capture. They capture the
environment *at the referencing expression*. That is, when I later say 'foo'
(for whatever reason: about to call, about to copy) the capture is made then.
The typestate system has to ensure that everything captured is init by the
time the capture takes place.



anonymous immediates locals w/ capture:
---------------------------------------

fn&(...) { ... }
fn~(...) { ... }
fn@(...) { ... }

capture occurs at the point of evaluation



type-inferencing abbreviation
------------------------------

{|x, y| ...}