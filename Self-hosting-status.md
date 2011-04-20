# Self-hosting status

## Typechecking

All modules typecheck.

## Translation

All modules translate (very slowly). However, verification fails:

    Instruction does not dominate all uses!
      %23 = load %str* %6
      store %str %23, %str* %8
    Instruction does not dominate all uses!
      %18 = load %str* %4
      store %str %18, %str* %9
    Broken module found, compilation aborted!
    Stack dump:
    0.	Running pass 'Function Pass Manager' on module 'rust_out'.
    1.	Running pass 'Module Verifier' on function '@_rust_fn1597_middle_typestate_check_print_ident'

