---
title: 'Stage 6: User defined types and Dynamic Memory Allocation'
---

# Stage 6: User defined types and Dynamic Memory allocation

!!! quote "Time estimate"
    2 weeks, 5-10 hours/week

!!! abstract "Prerequisites"
    1. Read the [ExpL specification](../expl.md).
    2. Read about [Dynamic memory allocation](../run_data_structures/heap.md).

!!! example "Learning Objectives"
    You will extend the language of Stage 5 by adding support for **user-defined types** and
    **dynamic memory allocation**. Issues of **Heap management** will be encountered en route.

This is the second major stage of the ExpL compiler project and will be implemented in two parts.
In the first part, we will see how user defined types can be added to the language syntax and how
semantic analysis can be performed. The ExpL specification demands storage for user-defined types
dynamically. We will discuss how dynamic memory allocation can be achieved in the second part.

See the [ExpL language specification](../expl.md) for an informal description of the language.
It is suggested that you design your own grammar using the outline
provided [here](../grammar-outline.md) as a reference.
The following [link](../testprograms/index.md#test-program-8--linked-list) provide examples
of ExpL programs containing user defined types.

We will now take up the front end - semantic analysis and AST representation - before proceeding
to code generation and dynamic memory allocation.


## **Part I**: Front End

Every user defined type requires a **type definition**. Type definitions are placed at the beginning
of a program, ahead of global declarations. A user-defined type in ExpL essentially defines
an **aggregate type**. The **member fields** of a user defined type may have arbitrary types
(subject to certain constraints – to be discussed soon).

Consider the type definition:

```
type
    bst {
        int a;
        bst left;
        bst right;
    }
endtype
decl
    int in,opt;
    bst insert(bst h, int key);
    int inOrder(bst h);
    int preOrder(bst h);
    int postOrder(bst h);
enddecl
```

This type definition specifies the node structure for a binary search tree. The member field a has 
type integer, whereas _left_ and _right_ have type bst. Note the **recursive** nature of the type 
definition. The declaration section shows functions which take as 
input a _bst_ or returns a _bst_. Here is another type definition:


```
type
    linkedList {
        int data;
        linkedList next;
    }
    markList{
        str name;
        linkedList marks;
    }
endtype
decl
    markList mList,temp;
enddecl
```

Note here that in the type _markList_, the member field _marks_ is of the type _linkedList_. 
ExpL stipulates that the member fields of a user defined type, if not of type integer or string,
can only be of the **same** type or one of a **previously defined** type.

The compiler must keep track of the type definitions in some data structure.
For this purpose, we will maintain a **type table** storing the type definition information.
**Each user defined type will have a type table entry**. In addition to user-defined types,
the type table will also store "default" entries for _int_, _str_, _bool_ and _void_ type.
(Since logical expressions evaluate to a boolean value, they may be assigned boolean type.
The ExpL constant NULL can be assigned to a variable of any user-defined type. Hence, having a
NULL type is useful from a purely implementation perspective. Note here that boolean is an
**implicit type** in the language. The language does not allow the programmer to declare a
variable of type boolean).

**The type table entry for a user-defined type must provide information about the names and
types of its member fields**. For each member field, a pointer to its type table entry must 
be maintained. This [link](../data_structures/type-table.md) gives you a simple type table
implementation scheme. (You have to fill in missing details).

Symbol table must also be modified to handle user-defined-types information. The **type field
of the symbol table entry of a variable/function** shall **refer to the type table entry of
the corresponding type** (Recall that in the case of a function, the type of a function is
its return type). The following [link](../data_structures/global-symbol-table.md) illustrates
the organization of the global symbol table. The type entry of each formal parameter of a
function must also refer to the corresponding type table entry.
[Local symbol tables](../data_structures/local-symbol-table.md) of functions will require similar modification.

The next question is how to assign memory for user defined type variables. We will defer this issue
temporarily and hence, for now, will not discuss how to assign bindings to variables of
user-defined types right now. This will be discussed in Part II.

We must now discuss how to use the symbol table and type table for completing semantic
analysis of the input program. Let us look at an example.

Consider the declaration of the type _markList_ in the example [above](#nav-marklist).
The language now permits statements like:

```
temp = mList.next;
if (mList.marks.data > mList.next.marks.data) then
    write("first student performed better in the first subject");
endif;
```

**Note that the operands in expressions can now be member fields of user-defined-type variables**.
Similarly, the left side of an assignment statement or the variable for a read statement can now be
a member field. The grammar rules for various statements in the language are outlined
[here](../grammar-outline.md). You must try to design your own grammar, keeping the grammar 
above as a guideline. Many details are (deliberately) left out in the outline given to you.

In the following, we use the term **field** generically to refer to _any_ member field of _any_ 
variable (of any user-defined type).

What must be the type of a field? The type of _mList.next_ in the above example must be the type
of the member field _next_ of the user defined type _markList_. Once this information is extracted 
from the symbol table, the type of any statement, expression or variable can be determined correctly.
Thus, **an assignment statement is valid provided the types of the right side expression matches
with that of the left side variable**. _The only exception to this rule is that the constant
NULL can be assigned to any variable of any user-defined type._

Stated formally,

```
Field :: = Field '.' ID { $$.type = $3. type; }
        | ID '.' ID { $$.type = $3.type; }
```

While constructing the AST, the type entry for any field may be recursively computed using the 
formal rule noted above. This information can be used for effective semantic analysis. Arguments
to functions must also be type checked against formal parameters of their declaration.

A plausible tree node structure and associated functions for 
AST construction are described [here](../data_structures/abstract-syntax-tree.md).

!!! note
    ExpL specification demands a rigid type analysis by
    [name equivalence](https://en.wikipedia.org/wiki/Nominal_type_system). This differs from the
    more liberal [structural equivalance](https://en.wikipedia.org/wiki/Structural_type_system)
    in the C programming language.

With this background, the front end of the ExpL compiler can be completed.

!!! question "Task 1"
    Complete the syntax and semantic analysis and construct AST for ExpL language.
    ([Specification](../expl.md), [Grammar Outline](../grammar-outline.md))

## **Part II**: Back End


We will first discuss the underlying concepts before getting into the back-end implementation details.

The ExpL specification stipulates that the storage for a variable of a user-defined-type is allocated
through the [_Alloc()_](../expl.md#nav-user-defined-types) function. Each user-defined-type variable
in ExpL must store the **reference** to its actual memory store.
**The actual memory may be allocated** by _Alloc()_ when a call to the function is encountered **at run time**.

From an implementation point of view, **a variable of a user-defined-type must be designed to hold
the address of its actual memory store**. Whenever the _Alloc()_ function is invoked for the variable,
a memory region sufficient for holding all the member fields of the variable must be allocated
"somewhere" in memory and Alloc() must return the starting address of that memory region.
**The compiler must generate code to invoke _Alloc()_ and store the return value into the
variable**. Thus, **the variable will essentially store a pointer** to the memory region
allocated by _Alloc()_. This is easy to do, provided we have the _Alloc()_ function at our disposal.

**A variable of a user-defined-type must be allocated at compile time one memory word to store an
address (basically an integer) returned by _Alloc()_ at run time**.
The allocation could be static or run-time. ExpL specification does not permit arrays of user-defined type.

Once the memory is allocated for a user-defined type variable, member field references can be 
translated easily. The details are left to you.

The next problem is to design and implement the _Alloc()_ function. We will also take up the 
issue of designing the _Free()_ function (to de-allocate some previously allocated memory). 
This problem is known as the **dynamic memory allocation problem**. (The _malloc()_ and _free()_
functions of the C library are [dynamic memory allocation](https://en.wikipedia.org/wiki/C_dynamic_memory_allocation)
routines of the C programming language.)

The strategy of Alloc() is to maintain a **memory pool** called **heap memory**. Alloc() will
need to manage a large run-time memory pool. To make matters simple, we will assume that Alloc()
divides the whole memory into fixed size blocks, each of size eight words.
In this case, the strategy is very simple:

1. Before the start of the program, reserve a large area of the address space for heap.
    The ExpOS [memory model](../abi.md#nav-virtual-address-space-model) suggests that the address
    region 1024-2047 may be used for this purpose.
2. Organize the heap into a linear linked list of blocks of size 8.
    We will design a heap initializer function _Initialize()_ specifically for this.
3. When an _Alloc()_ request comes, return the start address of the first
    free block in the list (and remove the block from the free list).
4. When a _Free(address)_ request comes (assuming _address_ refers to the start address of a
    block already allocated), return the block pointed to by _address_ back to the memory pool.

The _pragmatic_ restriction imposed by such a simple implementation is that a user-defined type
cannot have more than eight-member fields. Of course, we could have increased the block size to
– say sixteen – in which case the number of member fields can be upto 16.
For now, we will be content with the simple fixed block size scheme.

With this, we can design our compiler to generate code for _Alloc()_, _Free()_ and _Initialize()_
functions along with the target code. Techniques of Stage 5 suffices to build an executable file.

However, there is a better way of doing things when OS support for shared library is available.
Note that we have been using the shared library for console input, output etc. **Since the routines
_Alloc()_, _Free()_ and _Initialize()_ will be used by **every** ExpL program (that uses user-defined
types), it would be profitable to write these routines once and add them as part of the library.**
Since the OS loader loads the shared library to the memory region 0-1023 of the address space of
each program, the code for _Alloc()_, _Free()_ and _Initialize()_ too will be available to the
program. The library interface must be defined so that the calling conventions for
invoking each of the above functions are clearly specified.

With this strategy, when an ExpL program is compiled, the compiler will not generate code to
implement _Alloc()_, _Free()_ or _Initialize()_. **Instead, the compiler will generate a CALL
to the library (using the library interface) with appropriate arguments so that the corresponding
library routine is invoked. You need to modify the library code so that the assembly code for
_Alloc()_, _Free()_ and _Initialize()_ are added to the library.** The advantage of this method
is that the code implementing _Alloc()_, _Free()_ and _Initialize()_ need not be part of the the
code of every executable program, saving both load-time and system memory.
The technical jargon calls such a library a run time loading library.

The [ABI](../abi.md) stipulates that calls to the dynamic memory functions shall be directed
through the Library. Thus, you must add Alloc(), Free() and Initialize() as library functions.
As noted previously, the region of memory between address 1024 and 2047 must be used for heap memory allocation.

It is **absolutely necessary** to read and understand [Dynamic memory allocation](../run_data_structures/heap.md)
(**except** Buddy System Allocation) to proceed further. An overall picture of the ExpOS library design
is outlined in the [library implementation](../library-implementation.md) documentation.
We are now ready to complete the back end.

!!! question "Task 2"
    Complete the back-end adding _Alloc()_, _Free()_ and _Initialize()_ functions to the ExpOS
    library and complete the implementation of adding user defined types to ExpL. Assume fixed block
    size of 8 words for memory allocation. The compiler must flag an error "too many member fields"
    if a user-defined type definition has more than 8 member fields. Note that the fixed block
    allocator is pretty simple to be written directly in assembly language.
    Read the note below before proceeding with the implementation.

!!! note "Important Note"
    Your library functions will need to modify registers and hence before a library call,
    ensure that registers in current use are saved in the stack
    (as was done with function calls in the previous stage).

!!! questiosn "Exercise 1"
    Extend ExpL to permit arrays of user-defined type.

If you want to do **variable sized block allocation**, more complex allocation schemes like the
[**Buddy System Algorithm**](../run_data_structures/heap.md#nav-buddy-allocation) will be required.
One would also need to understand the issue of
[**memory fragmentation**](https://en.wikipedia.org/wiki/Fragmentation_(computing))
that can arise when variable sized allocation is done.

!!! question "Exercise 2: (Optional)"
    Modify _Alloc()_ and _Free()_ library functions to implement the Buddy memory allocator
    described [here](../run_data_structures/heap.md#nav-buddy-allocation). You will have to modify
    _Initialize()_ appropriately. The buddy system allocator is too complex to write in assembly
    language. Hence write them in ExpL itself and modify your label translation scheme
    to generate target addresses correctly.

## Test Programs

Check your implementation with the following test cases:

- [Test Program 1: Linked List](../testprograms/index.md#test-program-8--linked-list)
- [Test Program 2: Binary Search Tree](../testprograms/index.md#test-program-9--binary-search-tree)
- [Test Program 3: Extended Euclid Algorithm using linkedlist](../testprograms/index.md#test-program-11-extended-euclid-algorithm-using-linkedlist)
- [Test Program 4: Extended Euclid Algorithm using Userdefined types](../testprograms/index.md#test-program-4--extended-euclid-algorithm-recursive)

