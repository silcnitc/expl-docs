High Level Library Interface For expl   

[![](img/logo.png)](index.html)

*   [Home](index.html)
*   [About](about.html)
*   [Roadmap](roadmap.html)
*   [Documentation](documentation.html)

* * *

  

High Level Library Interface For expl
-------------------------------------

The High Level Library Interface is a unified interface to access system call routines and dynamic memory management functions from application programs.

Library Function / System Call

Function Code

Argument 1

Argument 2

Argument 3

Return Values

Read

"Read"

File Descriptor (int)

Buffer (int/str)\*

\-

0 - Success

\-1 - File Descriptor given is invalid

\-2 - File pointer has reached the end of file

Write

"Write"

File Descriptor (int)

Buffer (int/str)\*

\-

0 - Success

\-1 - File Descriptor given is invalid

\-2 - No disk space

\-3 - Permission denied

Initialize

"Initialize"

\-

\-

\-

0 - Success

\-1 - Failure

Alloc

"Alloc"

Size (int)

\-

\-

Address in the heap allocated (int)

\-1 - No allocation

Free

"Free"

Pointer (int)

\-

\-

0 - Success

\-1 - Failure

\* foot note regarding buffer

The description of the system calls can be seen here. The dynamic memory management routines are described below.

Dynamic Memory Routines
-----------------------

#### Initialize

Arguments: None

Return Value:

0

Success

\-1

Failure

  

Description: Intitalizes the heap data structures and sets up the heap area of the process.It is the applications responsibility to invoke Initialize() before the first use of Alloc(). The behaviour of Alloc() and Free() when invoked without an Intialize() operation is undefined. Any memory allocated before an Intialize() operation will be reclaimed for future allocation.

#### Alloc

Arguments: Size (int)

Return Value:

integer

Address in the heap allocated

\-1

No allocation

  

Description: The Alloc operation takes as input an integer, allocates contiguous words equal to the input specified and returns a pointer (i.e., an integer memory address) to the beginning of the allocated memory in the heap.

#### Free

Arguments: Pointer (int)

Return Value:

0

Success

\-1

Failure

  

Description: The Free operation takes a pointer (i.e., an integer memory address) of a previously allocated memory block and returns it to the heap memory pool. If the pointer does not correspond to a valid reference to the beginning of a previously allocated memory block, the behaviour of Free is not defined.

*   [Github](http://github.com/silcnitc)
*   [![Creative Commons License](img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)

Contributed By : [Thallam Sai Sree Datta](https://www.linkedin.com/in/dattathallam), [N Ruthvik](https://www.linkedin.com/in/n-ruthviik-0a0539100)

*   [Home](index.html)
*   [About](about.html)

  

window.jQuery || document.write('<script src="js/jquery-1.7.2.min.js"><\\/script>')