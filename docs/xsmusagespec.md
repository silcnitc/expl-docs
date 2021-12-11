---
title: 'XSM SIMULATOR USAGE SPECIFICATION'
hide:
    - toc
---

# XSM SIMULATOR USAGE SPECIFICATION

## Usage

The **XSM (eXperimental String Machine)** Simulator is used to simulate the hardware and OS abstractions specified in the ExpOS [Application Binary Interface](abi.md).

Within your XSM directory, use the following command to run the simulator

```bash
./xsm [-l library.lib] -e <filename.xsm> [--debug]
```

1. ***Syntax*** : `-l library.lib`

    ***Semantics*** : This flag loads the library, library.lib to the machine memory. (The simulator specification does not allow any name other than library.lib for the library file.) The argument is optional and needs to be given only if the library is to be linked to page 0 and page 1 of the [virtual address space](abi.md#nav-virtual-address-space-model)

2. ***Syntax*** : `-e <filename.xsm>`

    ***Semantics*** : This flag loads the executable file named as filename which is of the [XEXE format](abi.md#nav-XEXE-executable-file-format) . This argument is mandatory. The file is loaded into pages 4,5,6 and 7 in the [virtual address space](abi.md#nav-virtual-address-space-model) .

3. ***Syntax*** : `--debug`

    ***Semantics*** : This flag sets the machine into DEBUG mode when it encounters a BRKP machine instruction. Any [BRKP instruction](abi.md#nav-debug) in the program will be ignored by the machine if this flag is not set. Further details are given in the section below.

## Debugging

The ``--debug`` flag is used to debug the running machine. When this flag is set and the machine encounters a breakpoint instruction, the machine enters the DEBUG mode. In this mode a prompt is displayed which allows the user to enter commands to inspect the state of the machine. The commands in DEBUG mode are :

1. ***Syntax*** : `step` / `s`

    ***Semantics*** : The execution proceeds by a single step.

2. ***Syntax*** : `continue` / `c`

    ***Semantics*** : The execution proceeds till the next breakpoint (BRKP) instruction.

3. ***Syntax*** : `reg` / `r`

    ***Semantics*** : Displays the contents of all the machine registers namely IP, SP, BP, PTBR, PTLR, EIP, EC, EPN, EMA, R0-R19 in that order.

4. ***Syntax*** : `reg <register_name>` / `r <register_name>`

    ***Semantics*** : Displays the contents of the specified register.

    Sample usage: `r R5`, `reg SP`

5. ***Syntax*** : `reg <register_name_1> <register_name_2>` / `r <register_name_1> <register_name_2>`

    ***Semantics*** : Displays the contents of the registers from `<register_name_1>` to `<register_name_2>` in the order specified in (3).

6. ***Syntax*** : `mem <page_num>` / `m <page_num>`
    ***Semantics*** : Displays the contents of the memory page in the virtual address space.
    Sample usage: `mem 5`, `m 8`

7. ***Syntax*** : `mem <page_num_1> <page_num_2>` / `m <page_num_1> <page_num_2>`

    ***Semantics*** : Displays the contents of the memory from pages to in the virtual address space.

    Sample usage: `mem 4 7`, `m 8 9`

8. ***Syntax*** : `list` / `l`

    ***Semantics*** : List 10 instructions before and after the current instruction.

9. ***Syntax*** : `stacktrace` / `st`

    ***Semantics*** : List the contents of the top 12 memory locations on the stack, starting with the contents of the address pointed to by the SP register.

10. ***Syntax*** : `val <virtual_address>` / `v <virtual_address>`

    ***Semantics*** : Displays the content at memory address.

11. ***Syntax*** : `watch <virtual_address>` / `w <virtual_address>`

    ***Semantics*** : Sets a watch point to this address. Watch point is used to track changes of a particular memory location. Whenever a word which is watched is altered, program execution is stopped and the debug interface is invoked. Atmost 16 watch points can be set.

12. ***Syntax*** : `watchclear` / `wc`

    ***Semantics*** : Clears all the watch points.

13. ***Syntax*** : `exit` / `e`

    ***Semantics*** : Exits the debug prompt and halts the machine.

14. ***Syntax*** : `help` / `h`

    ***Semantics*** : Displays commands.
