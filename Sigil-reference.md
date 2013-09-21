A quick reference of the sigils and their meaning:

| Sigil | Meaning |
| ----- | ------- |
|  `~`  | Owned pointer; like a normal value, but always pointer-sized, and allocated on the heap |
|  `&`  | Borrowed pointer; a pointer to data that you do not own and thus cannot free |
| `&mut`| Mutable borrowed pointer; like a borrowed pointer, but cannot alias with any other pointer, and you can mutate through it |
|  `@`  | Managed pointer; a pointer to data that no one owns. Allows cycles. Garbage collected |
| `@mut`| Mutable managed pointer; a mutable pointer to data that no one owns. Has runtime checks to make sure it is not mutated at the same time by two different pieces of code, and that when it is borrowed, the guarantees of borrowed pointers are maintained |
|  `'`  | Lifetime/label marker; used to label loops for labeled break and to indicate the lifetime of a borrowed pointer (eg `&'a T`) |
|  `*`  | Raw pointer; just like a pointer in C. Requires unsafe code to dereference. Can be NULL |