High Level Library Interface For eXpOS   

[![](img/logo.png)](index.html)

*   [Home](index.html)
*   [About](about.html)
*   [Roadmap](roadmap.html)
*   [Documentation](documentation.html)

* * *

  

High Level Library Interface For eXpOS
--------------------------------------

The High Level Library Interface is a unified interface to access system call routines and dynamic memory management functions from application programs. The ExpL language allows applications to access the OS routines only through the library interface. The syntax for the call to the library function in ExpL is :

                              t = exposcall(fun\_code, arg1, arg2, arg3);
                          

Depending on the fun\_code the control is transferred to the system call routines (see below) .

Library Function / System Call

Function Code

Argument 1

Argument 2

Argument 3

Return Values

Create

"Create"

File Name (str)

Permission (int)

\-

0 - Success

\-1 - No Space for file

\-2 - If the file already Exists

Open

"Open"

File Name (str)

\-

\-

File Descriptor (int) - Success

\-1 - fFile not found or file is not a data or root file

\-2 - Process has reached its limit of resources

\-3 - System has reached its limit of open files

Close

"Close"

File Descriptor (int)

\-

\-

0 - Success

\-1 - invalid File descriptor

\-2 - File locked by calling process

Delete

"Delete"

File Name (str)

\-

\-

0 - Success

\-1 - File not found or file is not a data file

\-2 - File is open

\-3 - Permission denied

Write

"Write"

\-2

Buffer (int/str)\*

\-

0 - Success

\-1 - File Descriptor given is invalid

Seek

"Seek"

File Descriptor (int)

Offset (int)

\-

0 - Success

\-1 - File Descriptor given is invalid

\-2 - Offset value moves the file pointer to a position outside the file

Read

"Read"

File Descriptor (int)

Buffer (int/str)\*

\-

0 - Success

\-1 - File Descriptor given is invalid

\-2 - File pointer has reached the end of file

Fork

"Fork"

\-

\-

\-

PID (int) - Success, the return value to the parent is the process descriptor(PID) of the child process.

0 - Success, the return value to the child.

\-1 - Failure, Number of processes has reached the maximum limit. Returns to the parent

Exec

"Exec"

File Name (str)

\-

\-

\-1 - File not found or file is of invalid type

\-2 - Out of memory or disk swap space

Exit

"Exit"

\-

\-

\-

\-

Getpid

"Getpid"

\-

\-

\-

Process Identifier (int) - Success

Getppid

"Getppid"

\-

\-

\-

Parent Process Identifier (int) - Success

Wait

"Wait"

Process Identifier (int)

\-

\-

0 - Success

\-1 - Given process identifier is invalid or it is the pid of the invoking process.

Signal

"Signal"

\-

\-

\-

0 - Success

FLock

"FLock"

File Descriptor (int)

\-

\-

0 - Success

\-1 - File Descriptor is invalid

\-2 - Permission denied

FUnLock

"FUnLock"

File Descriptor (int)

\-

\-

0 - Success

\-1 - File Descriptor is invalid

\-2 - File was not locked by the calling process

Semget

"Semget"

\-

\-

\-

SEMID (int) - Success, returns a semaphore descriptor(SEMID)

\-1 - Process has reached its limit of resources

\-2 - Number of semaphores has reached its maximum

Semrelease

"Semrelease"

Semaphore Descriptor (int)

\-

\-

0 - Success

\-1 - Semaphore Descriptor is invalid

SemLock

"SemLock"

Semaphore Descriptor (int)

\-

\-

0 - Success or the semaphore is already locked by the current process

\-1 - Semaphore Descriptor is invalid

SemUnLock

"SemUnLock"

Semaphore Descriptor (int)

\-

\-

0 - Success

\-1 - Semaphore Descriptor is invalid

\-2 - Semaphore was not locked by the calling process

Shutdown

"Shutdown"

\-

\-

\-

\-

Newusr

"Newusr"

User name

Password

\-

0 - Success

\-1 - User already exists

\-2 - Permission denied

Remusr

"Remusr"

User name

\-

\-

0 - Success

\-1 - User does not exist

\-2 - Permission denied

Setpwd

"Setpwd"

User name

New Password

\-

\-1 - Unauthorised attempt to change password

Getuname

"Getuname"

User ID (int)

\-

\-

\-1 - Invalid UserID

User Name - Success

Getuid

"Getuid"

User name

\-

\-

\-1 - Invalid Username

uid - Success

Login

"Login"

User name (str)

Password (str)

\-

\-1 - Invalid username or password

\-2 - Permission denied

Test

"Test"

Unspecified

Unspecified

Unspecified

\-

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

\* Note: The Read() and Write() library functions expect a pointer to a memory address from (or to) which read or write is performed.

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