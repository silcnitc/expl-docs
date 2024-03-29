---
title: 'High Level Library Interface For eXpOS'
hide:
    - toc
---

# High Level Library Interface For eXpOS

The High Level Library Interface is a unified interface to access system call routines and dynamic memory management functions from application programs. The ExpL language allows applications to access the OS routines only through the library interface. The syntax for the call to the library function in ExpL is :

```
t = exposcall(fun_code, arg1, arg2, arg3);
```

Depending on the `fun_code` the control is transferred to the system call routines (see below) .

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
<td rowspan="3">Create</td>
<td rowspan="3"> "Create"</td>
<td rowspan="3">File Name (str)</td>
<td rowspan="3">Permission (int)</td>
<td rowspan="3">-</td>
<td>0  - Success</td></tr>
<tr><td>-1 - No Space for file</td></tr>
<tr><td>-2 - If the file already Exists</td></tr>

<tr>
<td rowspan="4">Open</td>
<td rowspan="4">"Open"</td>
<td rowspan="4">File Name (str)</td>
<td rowspan="4">-</td>
<td rowspan="4">-</td>
<td>File Descriptor (int) - Success</td></tr>
<tr><td>-1 - fFile not found or file is not a data or root file</td></tr>
<tr><td>-2 - Process has reached its limit of resources</td></tr>
<tr><td>-3 - System has reached its limit of open files</td></tr>

<tr>
<td rowspan="3">Close</td>
<td rowspan="3">"Close"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - invalid File descriptor</td></tr>
<tr><td>-2 - File locked by calling process</td></tr>

<tr>
<td rowspan="4">Delete</td>
<td rowspan="4">"Delete"</td>
<td rowspan="4">File Name (str)</td>
<td rowspan="4">-</td>
<td rowspan="4">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File not found or file is not a data file</td></tr>
<tr><td>-2 - File is open</td></tr>
<tr><td>-3 - Permission denied</td></tr>

<tr>
<td rowspan="2">Write</td>
<td rowspan="2"> "Write"</td>
<td rowspan="2">-2</td>
<td rowspan="2">Buffer (int/str)<font color="red">*</font></td>
<td rowspan="2">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor given is invalid</td></tr>

<tr>
<td rowspan="3">Seek</td>
<td rowspan="3">"Seek"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">Offset (int)</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor given is invalid</td></tr>
<tr><td>-2 - Offset value moves the file pointer to a position outside the file</td></tr>

<tr>
<td rowspan="3">Read</td>
<td rowspan="3">"Read"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">Buffer (int/str)<font color="red">*</font></td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor given is invalid</td></tr>
<tr><td>-2 - File pointer has reached the end of file</td></tr>

<!--- -->

<tr>
<td rowspan="3">Fork</td>
<td rowspan="3">"Fork"</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>PID (int) - Success, the return value to the parent is the process descriptor(PID) of the child process.</td></tr>
<tr><td>0 - Success, the return value to the child.</td></tr>
<tr><td>-1 - Failure, Number of processes has reached the maximum limit. Returns to the parent</td></tr>

<tr>
<td rowspan="2">Exec</td>
<td rowspan="2">"Exec"</td>
<td rowspan="2">File Name (str)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>-1 - File not found or file is of invalid type</td></tr>
<tr><td>-2 - Out of memory or disk swap space</td></tr>

<tr>
<td>Exit</td>
<td>"Exit"</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>

<tr>
<td>Getpid</td>
<td>"Getpid"</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>Process Identifier (int) - Success</td>
</tr>

<tr>
<td>Getppid</td>
<td>"Getppid"</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>Parent Process Identifier (int) - Success</td>
</tr>

<tr>
<td rowspan="2">Wait</td>
<td rowspan="2">"Wait"</td>
<td rowspan="2">Process Identifier (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - Given process identifier is invalid or it is the pid of the invoking process.</td></tr>

<tr>
<td>Signal</td>
<td>"Signal"</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>0 - Success</td>
</tr>

<tr>
<td rowspan="3">FLock</td>
<td rowspan="3">"FLock"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor is invalid</td></tr>
<tr><td>-2 - Permission denied</td></tr>

<tr>
<td rowspan="3">FUnLock</td>
<td rowspan="3">"FUnLock"</td>
<td rowspan="3">File Descriptor (int)</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - File Descriptor is invalid</td></tr>
<tr><td>-2 - File was not locked by the calling process</td></tr>

<tr>
<td rowspan="3">Semget</td>
<td rowspan="3">"Semget"</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>SEMID (int) - Success, returns a semaphore descriptor(SEMID)</td></tr>
<tr><td>-1 - Process has reached its limit of resources</td></tr>
<tr><td>-2 - Number of semaphores has reached its maximum</td></tr>

<tr>
<td rowspan="2">Semrelease</td>
<td rowspan="2">"Semrelease"</td>
<td rowspan="2">Semaphore Descriptor (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - Semaphore Descriptor is invalid</td></tr>

<tr>
<td rowspan="2">SemLock</td>
<td rowspan="2">"SemLock" </td>
<td rowspan="2">Semaphore Descriptor (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>0 - Success or the semaphore is already locked by the current process</td></tr>
<tr><td>-1 - Semaphore Descriptor is invalid</td></tr>

<tr>
<td rowspan="3">SemUnLock</td>
<td rowspan="3">"SemUnLock"</td>
<td rowspan="3">Semaphore Descriptor (int)</td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - Semaphore Descriptor is invalid</td></tr>
<tr><td>-2 - Semaphore was not locked by the calling process</td></tr>

<tr>
<td>Shutdown</td>
<td>"Shutdown"</td>
<td>-</td>
<td>-</td>
<td>-</td>
<td>-</td>
</tr>
<tr>
<td style="color:red" rowspan="3">Newusr</td>
<td rowspan="3">"Newusr"</td>
<td rowspan="3">User name</td>
<td rowspan="3">Password</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - User already exists</td></tr>
<tr><td>-2 - Permission denied</td></tr>

<tr>
<td style="color:red" rowspan="3">Remusr</td>
<td rowspan="3">"Remusr"</td>
<td rowspan="3">User name </td>
<td rowspan="3">-</td>
<td rowspan="3">-</td>
<td>0 - Success</td></tr>
<tr><td>-1 - User does not exist</td></tr>
<tr><td>-2 - Permission denied</td></tr>

<tr>
<td style="color:red">Setpwd</td>
<td>"Setpwd"</td>
<td>User name</td>
<td>New Password</td>
<td>-</td>
<td>-1 - Unauthorised attempt to change password</td>
</tr>
<tr>
<td style="color:red" rowspan="2">Getuname</td>
<td rowspan="2">"Getuname"</td>
<td rowspan="2">User ID (int)</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>-1 - Invalid UserID</td></tr>
<tr><td>User Name - Success</td></tr>

<tr>
<td style="color:red" rowspan="2">Getuid</td>
<td rowspan="2">"Getuid"</td>
<td rowspan="2">User name</td>
<td rowspan="2">-</td>
<td rowspan="2">-</td>
<td>-1 - Invalid Username</td></tr>
<tr><td>uid - Success</td></tr>

<tr>
<td style="color:red" rowspan="2">Login</td>
<td rowspan="2">"Login"</td>
<td rowspan="2">User name (str)</td>
<td rowspan="2">Password (str)</td>
<td rowspan="2">-</td>
<td>-1 - Invalid username or password</td></tr>
<tr><td>-2 - Permission denied</td></tr>

<tr>
<td>Test</td>
<td>"Test"</td>
<td>Unspecified</td>
<td>Unspecified</td>
<td>Unspecified</td>
<td>-</td>
</tr>
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

!!! note
    The Read() and Write() library functions expect a pointer to a memory address from (or to) which read or write is performed.

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
