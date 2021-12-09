---
title: 'Stage 7: Adding Objects – Data encapsulation'
---

# Stage 7: Adding Objects – Data encapsulation

!!! quote "Time estimate"
    2 weeks, 5-10 hours/week

!!! abstract "Prerequisites"
    1. Read the [OExpL specification](oexpl-specification.html) (except for the section on inheritance, which may be read before the next stage).

!!! example "Learning Objectives"
    You will extend the language of Stage 6 by adding support to classes (except for inheritance) to make ExpL an [_object based language_](https://en.wikipedia.org/wiki/Object-based_language).

In this stage, we will extend ExpL to provide support for [data encapsulation](https://en.wikipedia.org/wiki/Data_encapsulation) by supporting classes. [Inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)) will be added in the next stage.

From an implementation view-point, a class is similar to a user defined type that, apart from _member fields_, also contain _member functions_ or **methods**. The access rules of class members and methods are more stringent. Read the ([OExpL specification](oexpl-specification.html) carefully for access semantics.) Here, we will focus on how support for classes can be added to the compiler.


## **Part I**: Syntax and Semantic Analysis


The grammar outline for class definitions is given [here](oexpl-grammar-outline.html). The following grammar rule allowing class extension (in Class Definitions) will not be supported in this stage.

Cname : ID Extends ID {Cptr = Cinstall($1->Name,$3->Name);}

Support for class extension will be introduced in the next stage.

The compiler must maintain a **class table** to store the information pertaining to each class defined in an OExpL program. Class definitions appear after type definitions. Each class definition specifies the member fields and methods of the class. _Member field declarations must precede method declarations_.

For simplifying the implementation, **we will assume here that a class may contain at most 8 member fields and at most 8 methods**. Hence, fixed size allocation of dynamic memory will be sufficient, as in the 6th stage.

Each **class table entry stores information pertaining to a class**. The names and types of each member field along with its _position index_ must be stored in the class table (In any class, the method defined first will be assigned position index as **0**, the method defined next will be assigned position index as **1** and so on). For each method, the _signature_ of the method (method name, return type, types and names or arguments) along with the binding (_label_ of the method – a call to the method must be translated to a call to this label) needs to be stored. The type field for a variable/method must contain a pointer to the corresponding type table entry. _An exceptional case is when a member field is of a previously defined class. In this case, a pointer to the class table entry of the member field must be maintained_.

**A thorough reading of the details of [class table implementation](oexpl-data-structures.html) is necessary to proceed further**. In this stage, we will not support class extension. Hence, the parent class pointer entry for each class must be set to NULL.

For now, we will focus on syntax and semantics. Code generation will be taken up subsequently.

When **a variable of a class** is declared (in the global declaration section), **a pointer to the class table entry must be mainted in the [global symbol table entry](data_structures/global-symbol-table.html)** of the variable. Hence, a new _class table pointer_ field may be added to the global symbol table structure. Note that the [global symbol table entry](data_structures/global-symbol-table.html) for a global variable will have either a class table pointer entry or a type table pointer entry, but not both.

The following scope rules must be carefully checked to ensure correct semantic analysis:

*   1\. Methods of a class variable are accessed as _self.method\_name(..args..)_. (If a member is of a previously class, _self.field\_name.method\_name(..args..)_).
*   2\. **A member field in a class shall be accessed only within a method of the same class.** The access syntax will be _self.field\_name_. (**Note:** if _field\_name_ is variable of another class (or the same class), accessing member fields of the member using syntax _self.field\_name.sub\_field.name_ **is not be permitted – why?**)
*   3\. A method of one class is generally not permitted to access methods of other classes. However, if a class contains a member field of another class, then the methods accessible through the member field can be invoked as noted in point 1 above.
*   4\. There is **exactly one method carrying a name in a class**. Thus, [function overloading](https://en.wikipedia.org/wiki/Function_overloading) is not permitted.

These rules are not difficult to check once the class table is properly constructed from the class definitions. **All variable and function declarations visible to a method are contained in the class table entry of the relevant class**.

!!! question "Task 1"
    Complete the front end - lexical, syntax and semantic analysis for the extended language with classes.

## **Part II**: Code geneeration

Code generation is not hard, once the access semantics of _self_ is understood. **The basic observation is that the code for any method can be generated immediately as the definition of the method is processed**. This is true because a method's access is limited to its member fields and previously declared methods of the class. This information would be already entered into the class table entry for the class.

**When a method declaration is encountered, a unique label must be generated for the method and the label must be entered into the class table entry of the method.** Thus, the call address to any method can be determined from the class table entry of the corresponding class. (In the next stage, we will see that a more sophisticated strategy will be needed to support inheritance).

Storage allocation for class variables is equally simple. Just as was done for user defined types, a call to _'new'_ must allocate a block of 8 words in the heap for storing the member fields of the variable. The start address of this block must be assigned to the variable.

Other matters being routine, the non-trivial point pending is the **deferencing of the self reference** – in _self.field\_name_.

The following paragraphs summarize the key ideas:

*   1\. How to get the address of _self?_ First observe that a reference to _self_ will occur only within a member function of a class variable. Second, all variables of the same class share the code of all the methods of the class. Thus, at run time, a method must be told which is the variable for which the current call is made. In other words, **the address of self can be determined only at run-time (_why? - ensure that you digest this point clearly before proceeding further!_)**

    The standard way to resolve this reference is to set the convention that **before a call to a method, the caller must push the address of _self_** (for the particular call) **as an argument into the stack.** The convention we suggest is to push the address of _self_ before pushing the arguments during method invocations. Note that _self_ is an **implicit argument**, not found either in the declaration or the definition of the method. In the next stage, we will need to push one more implicit argument for each method invocation. It is instructive to have a quick look at the [run-time-stack management documentation](oexpl-run-data-structures.html#nav-runtimestackmanagementformethodinvocations) for OExpL at this stage


*   2\. The **local symbol table of a method must contain an entry for _self_** along with the other arguments to the function. The binding field of this entry must be the relative address (with respect to BP in the stack) where the address of self must be pushed by the caller. For example, if this value is -k, then the compiler expects that the caller would have pushed to stack corresponding to \[BP-k\] the address of the heap block holding the member fields of the variable.

The task of completing code generation phase is now straightforward.

!!! question "Task 2"
    Complete code generation for this stage.

!!! question "Exercise 1"
    What modifications must be done to allow class variables to be passed as arguments to functions?

!!! question "Exercise 2"
    What modifications must be done to allow functions to have locally declared variables of class?

## Test Programs

Check your implementation with the following test cases :

- [Test Program 1: Binary Search Tree using Classes](oexpl-testprograms.html#test1)
- [Test Program 2: Linked list in OExpL](oexpl-testprograms.html#test2)
- [Test Program 3: Sum of factorials](oexpl-testprograms.html#test3)
