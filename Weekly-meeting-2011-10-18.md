## Attending
Brian (sound not working), Dave, Marijn, Niko, Patrick

* Niko - status: working on x64 stuff (Dave forgot to listen closely)
* Marijn  - status: working on tutorial & bugfixing
* Patrick - pulled away for Firefox mobile for a month; will stay involved in discussion but not hacking
* Patrick - Brian status: working on covariant generic issues (which will break typestate with refinements) and something about C stacks
* Marijn - question: are we moving from iterators to just blocks?
* Patrick - I think for 0.1 that's what we want to do
* Dave - should we leave the code there and pref it off?
* Patrick - we have VCS, just eliminate it rather than maintaining unused code
* Marijn - are we planning for 0.1 to go w/ monolithic stdlib? eventually this will be a bad idea
* Patrick - eventually stuff like HTTP clients won't be useful for everyone
* Patrick - sure, and EBML is questionable, etc. eventually we can separate into crates; not pressing right now since stdlib is small; go ahead and move EBML to separate crate
* Dave - I'll try to make sure Vidyo is set up and working for everyone before next meeting
