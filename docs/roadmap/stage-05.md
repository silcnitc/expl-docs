---
title: 'Stage 5: Adding Functions'
hide:
    - toc
---

# Stage 5: Adding Functions


!!! quote "Time estimate"
    2 weeks, 5-10 hours/week

!!! abstract "Prerequisites"
    You must read the following documents before proceeding with this stage:

    1. The main page of the document [Run time allocation](run-data-structures.html).
    2. [Run time stack Allocation](run_data_structures/run-time-stack.html).

!!! example "Learning Objectives"
    You will extend the language of Stage 4 by adding functions with support for recursion. Addition of functions to the language requires handling **scope** of variables. Support for recursion demands **run-time storage allocation**. Only _integer_ and _string_ type variables will be supported.

This is the first major stage in the ExpL project. A skeletal outline of the syntax rules for defining the extension of the language of Stage 4 to support subroutines is as below. You are required to fill in rules required to complete the grammar. Note that variables may be only of type integer/string.

Program ::= GDeclBlock FdefBlock [MainBlock](grammar-outline.html)
        | GdeclBlock MainBlock
        | MainBlock

GdeclBlock ::= DECL GdeclList ENDDECL | DECL ENDDECL

GdeclList ::= GDeclList GDecl | GDecl
GDecl ::= Type GidList ;

GidList ::= GidList , Gid | Gid

Gid ::= ID
     | ID\[NUM\]
     | ID(ParamList)
\--------------------------------------------------------------------------------------
FDefBlock ::= FdefBlock Fdef | Fdef

Fdef ::=Type ID ( ParamList ) { LdeclBlock Body }

ParamList ::= ParamList , Param | Param
        |   /\*param can be empty\*/

Param ::= Type ID

Type ::= INT | STR
\----------------------------------------------------------------------------------------- LdeclBlock ::= DECL LDecList ENDDECL | DECL ENDDECL

LDecList ::= LDecList LDecl | LDecl

LDecl ::= Type IdList ;

IdList ::= IdList, ID | ID

Type ::= INT | STR

Since a function call is treated as an expression (whose value is the return value of the function), the following rules must be added:

E ::= ID () | ID(ArgList)

ArgList ::= ArgList, E | E

Here is an [example](run_data_structures/run-time-stack.html#nav-illustration) for a program with a function. We will take up semantic analysis and AST representation before proceeding to code generation.

Each function requires a **declaration**. The declaration of functions must be made along with the global declarations. The declaration of a function must specify the types and names of the **formal parameters** and the **return type** of the function. The compiler must store the declaration information in the global symbol table. For example, the declaration

decl
..
..
 int factorial(int n);
..
..
enddecl

specifies that _factorial_ is a function that takes as input one integer argument and returns an integer. This is sometimes called the **signature** of the function. Conceptually, to invoke the factorial function, the **caller** must know:

*   1\. The memory address to which the function call must be directed (**binding**).
*   2\. The **types and names of the formal parameters** to the function and the order in which the actual **arguments** must be given as input to the function.
*   3\. The **return type** of the function.

This precisely is the information that the symbol table stores.

A function **definition** contains:

*   a) The function's signature.
*   b) The declaration of **local variables** of the function.
*   c) The code of the function.

For example, the definition of the factorial function could be as:

int factorial(int n){
 decl
  int f;
 enddecl
 begin
  if( n==1 || n==0 ) then
   f = 1;
  else
   f = n \* factorial(n-1);
  endif;
 return f;
 end
}

Local variables declared in a function are _visible only_ within the function. We say that the **scope** of a local declaration is limited to the function. Moreover, if a global variable is redeclared inside a function, _the local declaration overrides the global declaration_.

Thus, we have two kinds of variables. Global variables that are visible "everywhere" (or having a **global scope**) and local variables that are visible only within the functions (or having a **local scope**) where they are declared.

The compiler needs to know the binding addresses and types of the local variables for translation of statements of the function to assembly code. However, this information is irrelevant outside the function.

To keep track of the local variable and scope information, our strategy is to keep global and local variables in different symbol tables. We will have

*   1\. A single **global symbol table** storing the _(name, type, size, binding)_ information of global variables as well as _(name, type, parameters, binding)_ information for functions. The following structure is suggested for storing a global symbol table entry.

    **struct Gsymbol{**
      **char \*name;** //name of the variable or function
      **int type;** //type of the variable:(Integer / String)
      **int size;** //size of an array
      **int binding;** //static binding of global variables
      **struct Paramstruct \*paramlist;**//pointer to the head of the formal parameter list
                       //in the case of functions
      **int flabel;** //a label for identifying the starting address of a function's code
      **struct Gsymbol \*next;** //points to the next Global Symbol Table entry
    **};**

*   2\. Several **local symbol tables** - one for each function containing the _(name, type, binding)_ information for each local variable of the function. (Note that since the language does not permit arrays to be defined within a function, the size value of local variables is always 1, hence the field is not required). A local symbol table entry can be stored in the following structure:

**struct Lsymbol{**
  **char \*name;**     //name of the variable
  **int type;**    //type of the variable:(Integer / String)
  **int binding;**     //local binding of the variable
  **struct Lsymbol \*next;**   //points to the next Local Symbol Table entry
**};**

**Note:** As noted in the [run time storage allocation](run-data-structures.html) documentation, local variables cannot be assigned static memory addresses. Hence, the binding of a local variable is a relative address within the function's activation record. We will discuss this matter in detail later.

What is the **stored in flabel**? When the compiler generates code for the function, a label is placed at the start of the function. A call to the function is translated to a low-level CALL to the corresponding label. Later, the label has to be replaced by the address of the instruction during the [label translation phase](label-translation.html).

The simple scheme we suggest here is to put the label F0 before the code of the first function declared in the program, F1 before the code of the second and so on. Hence, in the flabel field of the first function, store 0. Similarly, store 1 in the flabel field of the second function and so on. A call to the first function must be translated to CALL F0 and so on.

With this, the global symbol table for the [program](run_data_structures/run-time-stack.html#nav-illustration) would be as below.

![](img/gsymboltable3.png)

Continuing with the above example, we need two local symbol tables – one for the main function and one for the factorial function. The local symbol table holds the (name, type, binding) triple for each **formal parameter** as well as **local variables** of the function. We will discuss the binding of formal parameters and local variables later. The local symbol tables of main and factorial would look as the following:

![](img/localsymboltable2.png) ![](img/localsymboltable1.png)

See [LINK](data_structures/local-symbol-table.html) for more details. For now, ignore the type table pointer in the structure given in the link. This will be discussed in the next stage.

**Task 1**: Complete the program to build the global symbol table. Test your program by printing out all global declarations in the program by displaying the contents of the symbol table. You must not permit two variables/functions (or a function and a variable) to have the same name.

Our next aim is to perform semantic analysis, build the AST and then generate code. The strategy will be the following:

*   1\. First, parse global declarations and create the global symbol table entries for functions andglobal variables (Already completed as Task 1)
*   2\. For each function for which code is not yet generated
    *   a) Check for **name equivalence** of the formal parameters of the function definition with the declaration. Name equivalence requires that the type and name of each formal parameter of the function in the declaration and the definition must agree.
    *   b) Create local symbol table containing local variables and parameters.
    *   c) Build AST for the function. (Do type-checking when the tree is being built, as was done in the previous stages.)
    *   d) Recursively traverse the tree and generate code for the function in the target file.

**After generating code for a function, the local symbol table and the abstract syntax tree for the function can be deallocated.** Proceed to step 2 for the next function.

Note that the code and the local variables of a function are not visible outside the function. Hence, the local symbol table and the AST for a function are not required once the code is generated. **To generate code for calling one function from another function, only the global symbol table information of the callee is needed. The global symbol table is maintained throughout the compilation process.** Note that the main function is just like any other function except that it has no arguments, its name must be _main_ and return type must be _int_ as per the specification.

The only non-trivial part about semantic analysis and AST construction pending discussion is how to build AST for a function call.

We start with an example. Suppose we have the following declarations:

int Compute(int p, int q);
int find(int x);

A call to the function would occur as in:

t = Compute(a+b,find(a-b));

The statement is semantically valid provided a, b, and c integer type variables. Note that the call might occur in the code of _any function (including the same function – as in the case of a recursive call)_.

The expression tree for this could look as below:
![](img/expression tree.png)

A tree node for a function call contains a pointer to a list of expressions, one expression for each argument. **The compiler must type check each argument and match it with the type of the corresponding formal parameter of the called function.**

To get an overall picture of what is going on, you may read the documentation on [compile time data structures](data-structures.html) at this juncture. **Ignore type table entries for now** as we permit only integer/string variables in the present stage.

The expression tree structure given [HERE](data_structures/abstract-syntax-tree.html) can be used (again, ignore type table pointer) The details of implementation are left to you.

With this information, the task of completing type checking and building expression tree for a function is straightforward.

**Task 2**: Construct AST for each function after checking type and scope rules (semantic analysis). Right now, we are concerned just with type and scope analysis and not code generation. Note that a variable appearing in a function must first be searched for in the local symbol table of the function and then in the global symbol table, if not found in the local symbol table. Note that local declaration overrides global declaration. The compiler must report an error if the types, names and number of arguments in each function declaration are not matching with the function definition.

Now we turn to the code generation step.

The dynamics of a function call can be understood easily by dividing the process into the following three stages.

*   Step 1. **Code executed in the calling function (caller) before the transfer of control** to the called function (**callee**):
    *   a) The caller must **save the registers** in use so that even if the callee changes the values later, the original values can be recovered after return from the callee.
    *   b) The caller must **evaluate the arguments to the callee and store it** somewhere in a way that the **callee can access those values**.
    *   c) The caller must create some **space for the callee to place the return value** at the end of the call.
    *   d) The caller must **transfer control to the binding address of the callee**.
*   Step 2. **Actions executed by the called function (callee)**:
    *   a) The callee must **allocate space for its local variables**.
    *   b) Execute the callee code.
    *   c) When encountering a return instruction, the **return value must be computed and stored** in the appropriate place specified by the caller (step 1.c above).
    *   d) **Return** control to the caller.
*   Step 3. **Actions executed by the caller** after the callee returned control back to the caller.
    *   a) **Recover the return value** stored by the callee.
    *   b) **Deallocate space allocated for arguments** for the call in step 1(b).
    *   c) **Recover the machine registers** stored before the call in step 1(a).

The machine code for actions in Step 1 and Step 3 must be generated when the compiler encounters a function call in the caller's code. The compiler generates code for Step 2 while generating code for the callee function.

To implement the above plan, we need to create storage space whenever a function call is encountered. We will be focusing on generating code containing labels. Translation of labels to actual addresses can be easily done at the end as was done in Stage 4 following the [Label Translation Documentation](label-translation.html).

**Implementation Strategy**.

The fundamental strategy for space allocation is to create an **activation record** for the callee in the **stack** when a function call is encountered. **The compiler must generate code for creating the activation record at run-time when the call is encountered**. Since the storage requirements for arguments, local variables and the return value of a function are known at compile time, the compiler knows exactly how much space must be allocated for each function. We propose following general organization for activation records:

*   1\. Each **activation record must have a base (memory location)** which is determined at run time. The machine register **BP (base pointer)** is generally used to point to the base of the activation record of the function executing currently.
*   2\. **Relative to the base, the address of each argument, each local variable, the address where the return value is stored etc are fixed by the compiler statically** (at compile time).
*   3\. Initially, the activation record for the main function is created in the stack. BP is initialized to the base of this activation record and the main function starts execution.
*   4\. **If function A calls function B, a new activation record is created in the stack for function B above the activation record of function A**. The BP is made to point to the base of activation record of B. Upon return from B, the activation record of B is popped off the stack and BP is set back to the activation record of A.
*   5\. If function A calls function B, the address of the instruction in A to resume execution (**return address** – value of current-IP +2 in XSM machine- why?) upon return from B must be saved. Similarly, the **base pointer of the caller** (BP value) of A **must be saved in the stack** before BP is changed to point to the base of B. Both the return address and BP values will be stored in pre-defined locations of the activation record of B.
*   6\. In addition to the above, one additional **space** must be reserved in the activation record of B **to store the return value**.

A thorough reading of this [page](run_data_structures/run-time-stack.html) is **absolutely essential** to proceed any further. Suppose a function has n arguments (arg\_1, arg\_2,...,arg\_n) and m local variables (loc\_1, loc\_2, ..., loc\_m), its activation record in the stack may look as the following.(The stack is assumed to grow downwards.)

![](img/stack.png)

In this scheme, the following code must be generated by the caller when a call to the above function is encountered:

*   1\. Generate code to **push registers** in use into the stack. After this, the callee's activation record begins.
*   2\. Evaluate arg\_n and push the value to the stack. Now evaluate and push arg\_n-1 and so on till arg\_1. The **arguments are pushed in reverse order**. (You can do it in any order as long as the same convention is followed everywhere).
*   3\. **Push one empty space** in the stack for the callee to store the **return value**.
*   4\. Generate **Call instruction** to the binding (label) of the function. (The call instruction will push IP+2 into the stack and jump to label.)

Stack before the call instruction :
![](img/before call.png)


Stack after the call instruction :
![](img/after call.png)






Figure: Actions in the stack done by the caller before the call


**Once the call is made, the next instruction in the caller will be executed only after the callee executes a return statement**. The caller must proceed from here assuming that **the callee would have placed the return value** in the location in the stack designated for it. The caller must generate code to extract the return value and clean up the stack.

*   5\. Allocate a new register and **store the returned value** into the register.
*   6\. **Pop out arguments** from the stack. (The arguments may be discarded now.)
*   7\. **Restore registers** saved in the stack in step 1.

The above actions are sufficient to generate code for handling a function invocation. Note that to generate code for a call, only the callee's declaration information (argument information and call label) needs to be known. All that the caller needs to know is the callee's **interface**, and the **calling convention** (convention regarding in what order arguments must be pushed, where should the return value be stored etc. See also [link](https://en.wikipedia.org/wiki/Calling_convention)).

Now, we turn to the actions to be done by the callee. The callee must generate the following code before proceeding to the remaining instructions:

*   1.**Save the BP of the caller** by pushing the BP register into the stack.
*   2.**Set BP** to the present value of SP register.
*   3\. Push enough space in the stack for storing the local variables.

Relative to the BP value set in step 2 above, \[BP-2\] is the address to which the return value must be stored. \[BP-3\] stores arg\_1, \[BP-4\] stores arg\_2 and so on. \[BP+1\] is for loc\_1, \[BP+2\] for loc\_2 and so on. Thus, after seeing the local variable declarations, the compiler can set the binding values for local variables relative to the base of activation record (BP) value as:

![](img/var-bind table.png)

With this convention, the code for each instruction inside a function can be generated following the rule: **local variables/arguments are to be dereferenced a by adding the binding value to the contents of the BP register**. Almost every modern architecture supports function calls by providing an explicit base pointer register.

Finally, the code for a return statement must:

*   1\. **Pop out the local variables** from the stack.
*   2\. Calculate the return expression and store the value in \[BP-2\].
*   3\. **set BP to the old value** of BP in the stack.
*   4\. **Execute the RET instruction** to pass control back to the caller.


In the calling convention which we described above, the arguments were pushed in reverse order, space for the return value was allocated by the caller, BP of the caller was saved to the stack by the callee, and so on. Space created in the stack by either the callee or the caller must be eventually reclaimed by the same party.

**IMPORTANT NOTE**: In our calling convention, the caller was required to save the registers in use before the call. What if instead, we design a calling convention where the callee had to push the registers in use? The problem here is that the callee does not know (and does not have to know) the caller and hence do not know at each call which were the registers in use. Hence, the callee will have to save all machine registers, wasting time and space. Hence, the convention of the caller storing registers in use is superior. However, there are situations where this is not possible. For instance, in hardware interrupt routines, control is transferred to the callee without the caller executing a call. In such cases, the callee will have to save (all) the machine registers for a successful return.

You have enough background now to complete the final task of this stage.

**Task 3**: Complete code generation for functional calls.

**Exercise 1**: Modify the function semantics to permit pointer type variables of [Stage 4 (Exercise 2)](#stage4_ex2) to be passed as arguments to functions. This will allow a function to pass the address of a local variable to another function as an argument so that the callee can modify the contents. Modify the syntax and semantics rules appropriately. This feature must allow you to write functions like:

_int_ swap_(int_ \*p, _int_ \*q_)_

Note that functions need to be permitted to return pointer type variables. (Returning a pointer to local variable from a function is not advisable – Why?).

**Exercise 2** (Hard work, but insightful): Suppose you want to extend the language with facility of **tuples**. By a tuple, we mean an object declared as below:

decl
..
..
 tuple tnme(type fname\_1, type fname\_2, .... ,type fname\_n) var\_1, var\_2 .. var\_k;
..
..
enddecl

Note that tuple is introduced as a new keyword. For example, we could have:

decl

 tuple student (str name, int roll\_no, str branchname, int year\_of\_admission) a, b, c, \*sptr;
 tuple faculty (str name, int employee\_id, str dept) x, y, z, \*fptr;

enddecl

To access a tuple you must introduce the "." operator. Here is an example:

read(a.name);
read(x.name);
if (a.name == x.name) then
 write("They have same names");
endif;

You must also permit assignment of a tuple type variable to another, provided the variables are of the same tuple type.

Design the syntax and semantics rules, make necessary modifications to the lexer, parser, symbol table and AST structures to incorporate the addition of tuples and change the code generation module accordingly. For now, assume that tuples cannot be passed as arguments to functions or be returned by functions. However, you must permit local tuple declarations. Note that you have considerable freedom in deciding on the grammar rules and data structures, and even the features permitted.

**Exercise 3**: (Hard work, optional, but insightful. Can be done only after Exercise 1 and Exercise 2 are completed).
Allow tuples and pointers to tuples to be passed as arguments to functions. (Allowing the whole tuple to be passed creates more work in parameter passing, though not difficult in principle). Functions may be permitted to return tuples as return values. Permit a function that takes an argument/returns a tuple or pointers to tuple type to be declared only after the declaration of the concerned tuples (this is to avoid the [forward reference problem](https://en.wikipedia.org/wiki/Forward_declaration)). Design syntax, semantics and code generation strategies appropriately.

## Test Programs

Check your implementation with the following test cases :

- [Test Program 1 : Bubblesort (recursive)](testprograms.html#test3)
- [Test Program 2 : Factorial (recursive)](testprograms.html#test5)
- [Test Program 3 : Quicksort (recursive)](testprograms.html#test6)
- [Test Program 4 : Constant Program (recursive)](testprograms.html#test7)
- [Test Program 5 : Fibonacci (recursive)](testprograms.html#test10)
- [Test Program 6 : Extended Euclid(with a Function)](testprograms.html#test2)
- [Test Program 7 : BubbleSort (iterative)](testprograms.html#test1)
- [Test Program 8 : Extended Euclid(iterative)](testprograms.html#test12)

