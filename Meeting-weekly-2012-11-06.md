# Attending

brian, tim, graydon, patrick, niko

# Agenda

- floating point literal inference

# Buildbot

* G: Been fighting buildbot all week.  
* G: If people could actually start watching buildbot as well that would be nice but it might be too early for that
* B: What kind of failures are we seeing?
* G: It's configured to test a little bit more than Rustbot was.
* G: It is running both a full and quick incoming cycle.
* G: Quick is single target, no valgrind, check-lite
* G: Full does an all targets, all tests, valgrind build that kicks in from time to time
* B: Console is green for everything but windows, which seems pretty good
* G: "dist" target got broken but...
* B: ...at least we have dist testing now.
* G: Though not on a regular cycle
* G: Unfortunately buildbot seems to corrupt its workspace more often

# Licensing

* G: One option is MIT-APL dual license

# Floating point literal inference

* P: Floating point inference. It's come up several times. Not having it is annoying. Etc. Particularly for * graphics programming.
* G: Integer inference important for literal arguments. Very common.
* P: *Litany of graphics examples*.
* P, G, B, N: *Missed discussion. Will implement float inference.*

# Constants in patterns

* P: Constants in patterns. We detect when it happens and make it an error. Why don't we 'do the right thing'.
* N: Consts can't be shadowed, act like nullary variants?
* P: That is the situation today. Consts have all caps convention so shadowing isn't a big deal.
* All: *agreement*

# Moves based on type

* P: I'm going to experiment with implementing moves based on types rather than move/copy keywords* 