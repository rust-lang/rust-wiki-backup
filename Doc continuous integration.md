Rust is a fast-moving project and it can still be difficult for package maintainers to keep up with the Rust master branch. We recommend that maintainers use [Travis CI](https://travis-ci.com/) to test their packages against Rust nightly builds and register with the [Rust CI](http://hiho.io/rust-ci/) community dashboard. See the Rust CI page for further instructions.

First things first, you'll need to set up your project with Travis CI. Sign up for an account on their website, and then in Travis's profile page, make sure the project you want to check is enabled.

Next, you'll need to write a file `.travis.yml` with the single line `language: rust` in the root of your project and you are good to go! All commits and pull requests will launch a new test using `Cargo`.

See the [full documentation](http://docs.travis-ci.com/user/languages/rust/) on Rust support (for example, if you do not want to run `Cargo` to run tests).