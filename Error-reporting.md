# Fix-Its

`rustc` should output "fix-its" like clang does. Here are some ideas:

* Incorrect use of numeric literals.

        auto i = 0u;
        i += 3; // suggest "3u"

* Use of `for` where `for each` was meant.

        for (v in foo.iter()) // suggest "for each"

* Typos on record field names.

        fn f(rec(int foo, int bar) r) { ... }
        f(rec(foo=1, baz=2)); // suggest "bar"
