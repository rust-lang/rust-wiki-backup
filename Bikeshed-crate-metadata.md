This is a braindump of possible crate metadata either declared as attributes or otherwise encoded in a (currently hypothetical) packaging system.



Name | Accepted Values | Rationale
---  | ---             | ---
name | string          |
version | semantic version tuple |
long name/authority | fully-qualified name | should published names be first-come, first-served, or should a long name include provenance?
description | string | useful for an index of published crates
license | enumerated list |
manifest of files | list of filenames | helps to verify that a crate bundle is complete
author | string |
homepage / repository / bugtracker | URLs |
build / test / optional dependencies | list of crates | splitting dependences into categories helps dependency resolvers and packagers
cryptographic signature | crypto hash | offers some degree of protection against tampering, if signatures are distributed apart from crates
enabled / prohibited platforms | list of platform triplets | platform-specific crates, such as Win32 or POSIX or iOS or Android
stability | enumerated list | see proposal for library stability attributes
package format version | version number | useful to protect against changes to packaging format