If a published index of crate were to exist, what problems would it solve? What would it look like? What would it include? The equivalents in other languages include gems, npm, and CPAN.

## Who does it serve and what do they want?

Several distinct cohorts may exist:

 * embedded system developers
 * developers of standalone apps (a Servo-based browser, for example)
 * developers of free software
 * developers of paid proprietary software
 * developers of mobile applications
 * in-house developers
 * distribution packagers
 * end-users of software packaged by distributions

The mechanisms of using crates varies by cohort. Some prefer source. Others may not mind relying on binary distributions, whether proprietary libraries, free software packaged by others, or SDKs.

The upgrade strategy and frequency depends on cohort, but assume that the packaging system is flexible enough to allow multiple choices here.

## What questions should a packaging system answer?

 * what dependencies does a package have?
 * what versions are available for a package?
 * who is the authority for a package? (perhaps a package may have a short name and a long name)
 * which crates does the package include?
 * what license governs use (and distribution, modification, and redistribution)?
 * can the authority of a package be verified cryptographically?
 * is the package complete or has it been corrupted somehow?
 * does the package contain appropriate and well-formed metadata for the packaging system, or must it be extracted somehow? If so, can it be extracted accurately?
 * can the packaging system pin a package version locally?
 * can the packaging system add local patches to a package?
 * on which systems does the package work? On which systems does it not work? On which systems has it not yet been tested?
 * are there non-Rust dependencies (other dynlibs, platform-specific code)?
 * are there core dependencies? What are their stability levels?

## Open questions about a packaging system

 * should there be a mirroring system for packages?
  ** GitHub URLs are fine when GH is up
  ** GH URLs themselves may be fragile in the face of repository changes, branches, project renames, or forced pushes
  ** GH may be an inappropriate dependency for some cohorts
 * given a mirroring system, should there be a central index?
  ** who controls this?
  ** how is a package registered?
  ** how are names handled? name collisions?
  ** how is the index mirrored?
 * can packages be removed?
  ** copyright, trademark, and patent concerns may make this necessary
  ** security concerns may make this valuable
 * is there a standard metadata format? [[Bikeshed crate metadata]]
 * can ancillary tools interact with the packaging system?
 * are alternate packaging tools easy to write?
 * does the index system include old versions of packages?
 * what requirements does this place on the Rust stdlib?
 * is this a project separate from Rust itself, maintained out of the tree, or is it a necessary inclusion in Rust 1.0 or 2.0 or n.0?