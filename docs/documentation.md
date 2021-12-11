---
title: 'Documentation'
hide:
    - toc
---

# Documentation

Using LEX
---------

LEX is a tool used to generate a lexical analyzer.This document is a tutorial for the use of LEX for SIL Compiler development.Technically, LEX translates a set of regular expression specifications (given as input in input\_file.l) into a C implementation of a corresponding finite state machine (lex.yy.c). This C program, when compiled, yields an executable lexical analyzer..... [read more](lex.html)

Using YACC
----------

YACC (Yet Another Compiler Compiler) is a tool used to generate a parser.This document is a tutorial for the use of YACC to generate a parser for SIL.YACC translates a given Context Free Grammar (CFG) specifications (input in input\_file.y) into a C implementation (y.tab.c) of a corresponding push down automaton (i.e., a finite state machine with a stack).This C program when compiled, yields an executable parser.... [read more](yacc.html)

Using YACC with LEX
-------------------

In the previous documents, we have noted that YACC is used to generate a parser ([YACC documentation](yacc.html)) and LEX is used to generate a lexical analayzer ([LEX documentation](lex.html)). YACC generates the definition for yyparse() in y.tab.c and LEX generates the definition for yylex() in lex.yy.c. We have also noted that yyparse() repetitively calls yylex() to read tokens from the input stream....[read more](ywl.html)

ExpL Specification
------------------

This section of the document walks through the specification of compiler for Experimental Language(ExpL). The syntax and semantics are explained with general program structure and sample programs.The Data Types.... [read more](expl.html)

ExpL Grammar Outline
--------------------

An outline for the ExpL grammer is given. Calls to functions that update the symbol table, type table and the abstract syntax tree data structures are indicated as semantic actions at certain places.... [read more](grammar-outline.html)

OExpL Specification
-------------------

This section of the document walks through the specification of compiler for Object Experimental Language(OExpL). The syntax and semantics are explained with general program structure and sample programs. [read more](oexpl-specification.html)

OExpL Grammar Outline
---------------------

An outline for the OExpL grammer is given. Calls to functions that update the symbol table, type table and the abstract syntax tree data structures are indicated as semantic actions at certain places.... [read more](oexpl-grammar-outline.html)

Compile Time Data Structures for ExpL
-------------------------------------

The compilation of an ExpL program involves two phases. In the first phase (called the analysis phase), the source ExpL program is analyzed (lexical, syntax and semantic analysis are completed in this phase) and if the program is free of syntax and semantic errors, an intermediate representation of ... [read more](data-structures.html)

Compile Time Data Structures for OExpL
--------------------------------------

The compilation of an OExpL program involves two phases. In the first phase (called the analysis phase), the source OExpL program is analyzed (lexical, syntax and semantic analysis are completed in this phase) and if the program is free of syntax and semantic errors, an intermediate representation of ... [read more](oexpl-data-structures.html)

Run Time Data Structures for ExpL
---------------------------------

Before explaining the data structures used for the execution phase, it is necessary to understand the requirements and the underlying theoretical concepts in some detail. [read more](run-data-structures.html)

Run Time Data Structures for OExpL
----------------------------------

Before explaining the data structures used for the execution phase, it is necessary to understand the requirements and the underlying theoretical concepts in some detail. [read more](oexpl-run-data-structures.html)

Code Generation
---------------

ExpL compiler converts the intermediate representation(the abstract syntax tree) of the ExpL program to a machine code that can be readily executed by the XSM machine. The codeGen() function takes as input a pointer to the root of an abstract syntax tree and a pointer to a file to which the target code has to be written. The codeGen() function generates assembly code corresponding to the program represented by the AST. The codeGen() function essentially invokes itself **recursively** to generate ... [read more](codegen.html)

Label Translation (Linking)
---------------------------

Labels are generated while translating the high level program constructs such as if-then-else, while-do and function calls into target machine code as an intermediate step in code generation. Labels are needed because the target address of a JUMP instruction may not be known at the time of generating code. However these labels must be replaced .....[read more](label-translation.html)

Application Binary Interface
----------------------------

The ExpL compiler needs to translate a given source program and generate the target machine code into an executable file in a format which is recognized by the load module of the target operating system. Thus, in order to generate the executable, the following information needs to be made available to the compiler ...[read more](abi.html)

Library Implementation
----------------------

The ABI stipulates that the code for a **common shared library** must be linked to the address space of every program. The library code must be linked to logical page 0 and logical page 1 of each program. When an ExpL source program is compiled by an ExpL compiler, the compiler generates code containing calls to the library assuming the library functions will be "there" in the address space when the program is eventually loaded for execution. It is the responsibility of the OS loader to ...[read more](library-implementation.html)

XSM Simulator Installation
--------------------------

The installation instructions for the simulator can be found [here](install.html).

XSM Simulator Usage Instructions
--------------------------------

The XSM (eXperimental String Machine) Simulator is used to simulate the hardware and OS abstractions specified in the ExpOS Application Binary Interface. Within your XSM directory, use the following command to run the simulator ... [read more](xsmusagespec.html).

XSM Execution Environment Tutorial
----------------------------------

This tutorial helps you to gain a basic understanding of the execution environment provided by the XSM simulator. The compiler you design for the ExpL language is supposed to generate target XSM machine code that runs on the XSM Simulator provided to you. However, the bare machine cannot directly run the target code. The operating system (OS) that runs on top of the machine is the actual software that sets up an execution enviornment necessary for ... [read more](xsm-environment-tut.html).

ExpL Test Programs
------------------

The programs can be found [here](testprograms.html).

OExpL Test Programs
-------------------

The programs can be found [here](oexpl-testprograms.html).

* [Github](https://github.com/silcnitc)

[↑](#navtop "Go back up")

* [![Creative Commons License](img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)
* [Home](index.html)
* [About](about.html)
