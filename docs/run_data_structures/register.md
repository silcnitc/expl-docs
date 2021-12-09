---
title: 'Temporary Allocation'
hide:
    - toc
---

# Temporary Allocation

Registers are allocated and released for the temporary storage of intermediate computation through two simple functions.

- `#!c int get_register()`: Allocates a free register from the register pool (R0 - R19) and returns the index of the register, returns -1 if no free register is available.
- `#!c int free_register()`: Frees the last register that was allocated,returns 0 if success, returns -1 if the function is called with none of the registers being allocated.

