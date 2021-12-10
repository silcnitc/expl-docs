---
title: 'High Level Library Interface For expl'
hide:
    - toc
---

# High Level Library Interface For expl

The High Level Library Interface is a unified interface to access system call routines and dynamic memory management functions from application programs.

<div class="md-typeset__scrollwrap"><div class="md-typeset__table">
<table>
<tbody><tr>
<th>Library Function / System Call</th>
<th>Function Code</th>
<th>Argument 1</th>
<th>Argument 2</th>
<th>Argument 3</th>
<th>Return Values</th>
</tr>
<tr>
<td rowspan="3">Read</td>
<td rowspan="3">"Read"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">Buffer (int/str)<font color="red">*</font></td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor given is invalid</td></tr>
<tr><td>-2 - File pointer has reached the end of file</td></tr>
<tr>
<td rowspan="4">Write</td>
<td rowspan="4"> "Write"</td>
<td rowspan="4">File Descriptor (int)</td>
<td rowspan="4">Buffer (int/str)<font color="red">*</font></td>
<td rowspan="4">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor given is invalid</td></tr>
<tr><td>-2 - No disk space</td></tr>
<tr><td>-3 - Permission denied</td></tr>
<tr class="active">
<td rowspan="2">Initialize</td>
<td rowspan="2">"Initialize"</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - Failure</td></tr>
<tr class="active">
<td rowspan="2">Alloc</td>
<td rowspan="2">"Alloc"</td>
<td rowspan="2">Size (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>Address in the heap allocated (int) </td></tr>
<tr><td>-1 - No allocation</td></tr>
<tr class="active">
<td rowspan="2">Free</td>
<td rowspan="2">"Free"</td>
<td rowspan="2">Pointer (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - Failure</td></tr>
</tbody></table>
</div></div>

\* foot note regarding buffer

The description of the system calls can be seen here. The dynamic memory management routines are described below.

## Dynamic Memory Routines

### Initialize

Arguments: None

Return Value:

<div class="md-typeset__scrollwrap"><div class="md-typeset__table">
<table>
<tbody><tr>
<td>0</td>
<td>Success</td>
</tr>
<tr>
<td>-1</td>
<td>Failure</td>
</tr>
</tbody></table>
</div></div>
  

Description: Intitalizes the heap data structures and sets up the heap area of the process.It is the applications responsibility to invoke Initialize() before the first use of Alloc(). The behaviour of Alloc() and Free() when invoked without an Intialize() operation is undefined. Any memory allocated before an Intialize() operation will be reclaimed for future allocation.

### Alloc

Arguments: Size (int)

Return Value:

<div class="md-typeset__scrollwrap"><div class="md-typeset__table">
<table>
<tbody><tr>
<td>integer</td>
<td>Address in the heap allocated</td>
</tr>
<tr>
<td>-1</td>
<td>No allocation</td>
</tr>
</tbody></table>
</div></div>

Description: The Alloc operation takes as input an integer, allocates contiguous words equal to the input specified and returns a pointer (i.e., an integer memory address) to the beginning of the allocated memory in the heap.

### Free

Arguments: Pointer (int)

Return Value:

<div class="md-typeset__scrollwrap"><div class="md-typeset__table">
<table>
<tbody><tr>
<td>0</td>
<td>Success</td>
</tr>
<tr>
<td>-1</td>
<td>Failure</td>
</tr>
</tbody></table>
</div></div>

Description: The Free operation takes a pointer (i.e., an integer memory address) of a previously allocated memory block and returns it to the heap memory pool. If the pointer does not correspond to a valid reference to the beginning of a previously allocated memory block, the behaviour of Free is not defined.

