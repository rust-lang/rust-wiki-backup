This is a page of random performance ideas, mostly from pcwalton.

pcwalton:
graydon: there are a bunch of unnecessary allocations in trans

pcwalton:
I fixed a bunch of 'em but there are certainly more

pcwalton:
one thing that would very much help is a SmallVector in libstd

pcwalton:
fail() really needs to be a lang_item and it needs to be parameterized over ToStr

pcwalton:
the idea being that if you're failing with a constant string

pcwalton:
it should not call upcall_new_str_uniq() at the site of failure

pcwalton:
rather it should pass a string slice (&static/str) to a function which itself copies the string

pcwalton:
move the copy out of line

pcwalton:
likewise fmt! generates a ton of code. most of the time the unique string it produces is not really being used, except to be written once and thrown away

graydon:
mhm

pcwalton:
(1) stop zeroing stuff out to revoke cleanups, use a flag instead to indicate whether the dtor needs to be run (this is potentially a quite large win)

pcwalton:
(2) try to coalesce failure landing pads instead of duplicating the cleanups all the time

pcwalton:
also ty::sty comparison in the ty hashtable is calling into the shape glue. I'm going to try to fix that this week

pcwalton:
oh yeah, another thing I was thinking: stop making so many basic blocks

pcwalton:
especially when optimization is off, LLVM goes basic-block-by-basic-block in codegen

pcwalton: oh yeah, we should also see if we can get away with not zeroing out memory when we allocate it

pcwalton:
that's like 5% of our performance right there

pcwalton:
in some profiles anyway

## Memory moves

If you see large sequences of "mov" instructions, use `call_memmove` or `memzero` in trans, as appropriate. This is usually a symptom of code like `Store(bcx, Load(bcx, foo, bar), baz);`, which is *not* an efficient way to move large structural types around.