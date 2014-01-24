Code review checklist
---------------------

* Code should respect the policy described in the [style](https://github.com/mozilla/rust/wiki/Note-style-guide) page.
* Commit message summaries have to be descriptive.
* Commit messages should be *accurate*: if the commits have deviated from top-level description, don't `r+` without fixing the description (since bors will pull in the PR description as the commit message); see e.g. SHA:bf9c25562da1c0e768309693617e54e998a953d1 as an example of such deviation.
* Almost every change should contain a test case as described in the [testing](https://github.com/mozilla/rust/wiki/Doc-unit-testing) page.
* Code optimization should contain a bench case as described in the [bench](https://github.com/mozilla/rust/wiki/Doc-unit-testing#benchmarking) section of the [testing](https://github.com/mozilla/rust/wiki/Doc-unit-testing) page.
* Look for commits that could be squashed. 

General Suggestions
-------------------

* Don't do partial reviews. If you're reviewing a PR, address it completely. This will reduce the pending time of PRs.
* Whenever something can be improved or should be changed, be as detailed as possible in your comments. This will help contributors that are not familiar with the code to understand better what you're saying.
* Add references whenever it's possible. For instance, when a benchmark is requested, link the benchmark section to your comment, unless you're sure the contributor knows that already.

Non core contributors
---------------------

* If you reviewed a patch and code looks good to you, use `LGTM` instead of `r+`
