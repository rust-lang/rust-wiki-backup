## Just use Doxygen
lkuper, from email:

> Yes, one option is to patch Doxygen itself so it supports Rust directly: "If the grammar of X is close to C or C++, then it is probably not too hard to tweak src/scanner.l a bit so the language is supported. This is done for all other languages directly supported by doxygen (i.e. Java, IDL, C#, PHP)."  (http://www.stack.nl/~dimitri/doxygen/faq.html)

>The other option seems to be to write a filter that converts one's source code into something Doxygen can handle.  This has been done pretty often (http://www.stack.nl/~dimitri/doxygen/helpers.html), and Doxygen's config file format ("Doxyfile") includes support for an input filter that automatically runs over your stuff when Doxygen is run.

>Feel free to try out the Doxyfile I just pushed to https://github.com/lkuper/rust/tree/doxygen, if you're curious to see what happens when when one runs Doxygen on rustc right now.  It's under docs/ and all you should need to do is run 'doxygen' from there, assuming that Doxygen's installed.  The generated docs will end up in docs/html/ and docs/latex/.  The resulting documentation doesn't look like much, but we wouldn't expect it to, of course, since even if Doxygen did properly understand Rust code, we have no Doxygen-readable comments.  Because of that, I set the EXTRACT_ALL tag in the Doxyfile to YES to make Doxygen produce anything at all, but if we were actually writing Doxygen-readable comments then we'd probably want to set that to NO, to make Doxygen only produce documentation for stuff that's...actually documented.  I did have some luck just now with changing the comments in (for instance) trans.rs to Doxygen-readable form, but it's clear that just pretending that Rust is C++ won't fly.  :)

## Simple custom syntax 
Use `...` at the end to indicate that there should be out-of-band documentation as well. Because everyone likes Twitter, use `@` to refer to identifiers: either arguments (value or type), or other exported names, etc. `@vec` and other reserved words can be handled specially. Maybe backquotes will be used for code samples.

    /** Returns a @vec of @T, containing everything from @v 
     * whose index is in the range [@start, @end). ... */
    fn slice[T](array[T] v, uint start, uint end) -> vec[T] {
        assert (start <= end);
        assert (end <= len[T](v));

