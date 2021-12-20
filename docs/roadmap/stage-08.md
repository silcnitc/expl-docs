---
title: 'Stage 8: Inheritance and Sub-type Polymorphism'
---

# Stage 8: Inheritance and Sub-type Polymorphism

!!! quote "Time estimate"
    1 week, 5-10 hours/week

!!! abstract "Prerequisites"
    1. A rigorous reading of the [OExpL specification](../oexpl-specification.md)

!!! example "Learning Objectives"
    You will extend the language of Stage 7 by adding support for single inheritance and subtype
    polymorphism to make OExpL an _object oriented_ language.

In this stage, we will extend ExpL to provide support for 
[single inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)) and 
[subtype polymorphism](https://en.wikipedia.org/wiki/Subtyping). 
These features qualifies OExpL to be called an object oriented language.

Addition of support for inheritance and polymorphism involves some intellectual complexity. 
Fortunately, once the underlying conceptual issues are understood, the implementation is not difficult.

## **Part I**: Syntax and Semantics

The definition of a class could now be by extention of a previously defined class.

```
Cname : ID {Cptr = Cinstall($1->Name,NULL);}
      | ID Extends ID {Cptr = Cinstall($1->Name,$3->Name);}
```

An additional field called the **parent class pointer** must be added to each class table entry. 
If the class is defined by extention of another class, the parent class pointer entry must point
to the class table entry of the parent class. If the class is not an extention of any other class,
this entry must be NULL. ([See class table structure](../oexpl-data-structures.md)).
An outline of the grammar is given [here](../oexpl-grammar-outline.md).

An extended class inherits all the member fields and methods of the parent class.
The descendent class is permitted to do additional definitions as specified below.

1. The only member field declarations in the descendent class shall be for the new member fields
    that are not present in the parent. (**The descendent cannot re-declare or remove any of the parent's
    member fields.**) Additionally, in our current implementation plan, the number of member fields
    - including those inherited from the parent - must not exceed 8 (because our memory allocation
    policy allocates only 8 words for storing a class object). From an implementation perspective,
    **whenever the definition of a class by extension of another class is encountered, the compiler
    must copy all the member field information of the parent to the descendent class.** The member
    fields may be listed in the same order as they appear in the parent.
    Additional entries must be created for new members specific to the descendent class.

2. The descendent class may **define new methods** or **[override](https://en.wikipedia.org/wiki/Method_overriding)**
    one or more of the **parent's methods**. OExpL allows a class to contain only one definition for a function.
    **Once a method is over-ridden, the method of the parent class will no longer be visible inside the descendent class**.
    Moreover, the **signature of the overridden method must be identical with the signature of the method in
    the parent class** (in both the number and the types of arguments).
    In other words, [function overloading](https://en.wikipedia.org/wiki/Function_overloading)
    is not permitted. (Exercise 1 at the end asks you to relax this condition).

From an implementation perspective, the compiler may **initially copy the signatures of all methods of the parent
to the class table entry of the descendent**. New entries will have to be created for methods
that defined in the descendent, but not present in the parent. Finally, the present implementation
restricts the number of methods in a class ( including inherited methods ) to 8.

What about the labels of the methods of the descendent class? **If a method is over-ridden or defined
new in a class, the compiler must assign a new label for the method and store this label in the _Flabel_
field of the method in the [class table entry](../oexpl-data-structures.md). Otherwise, the class
inherits the parent's method and the label of the method in the parent class ( specified by the _Flabel_
field of the parent class ) must be copied into the _Flabel_ field of the method in the
descendent class** ( See Memberfunclist in the class table ).

Once the class table entries are created as described above, syntax and semantic analysis phases
of compilation can be completed easily.

An assignment of the form a = b; between class variables is valid if either _a_ and _b_ are
variables of the same class or _b_ is a variable of a descendent class of the class _a_. Similarly
`a=new(class_name);` is valid if the class identified by `class_name` is a descendent class of the
class to which _a_ belongs to. Thus, **a variable of a certain class can hold a reference to any
object of the same or descendent classes**. Since the class table contains adequate information to
perform this check, syntax and sematic analysis can be completed now.

**Task 1:** Complete the front end - lexical, syntax and Semantic analysis.

## **Part II**: Code Generation

The key differences between the language of Stage 7 and the present stage are the following:*

a. A variable of a parent class may hold a reference to an object of any descendent class.
b. The class to which the referred object belongs to at run time cannot be determined at compile time (why?)
c. When a method is invoked using the variable of the parent class, if the method is over-ridden by the child class, then the method of the child class must be invoked.

Assume that A, B, C are classes such that B extends A and C extends B. Let x be a variable of class A. Consider the program segment:

```
...
Read(n);
if (n>0) then
  x = new(B);
else
  x = new(C);
endif
  retval = x.fun();
...
```

Let `fun()` be a function defined in class A. Let us assume that the label for the function in the
class table is L1. Suppose that B does not over-ride _fun()_, but C over-rides _fun()_. 
Let the label for the over-ridden function be L2.

**Important Note :** Unless fun() is defined in class A, the call _x.fun()_ should result in a 
compilation error even if B and C contains the declaration for _fun()_. This is because if a 
variable of a parent class holds the reference to an object of a descendent class, only methods 
defined in the parent class are allowed to be invoked using the variable of the parent class.

The call _x.fun()_; must translate to CALL L1 if (n>0) whereas _x.fun()_ must translate 
to CALL L2 otherwise. The value of n is known only at run-time and the compiler can't hope to guess it.
What will the compiler do now?!

It turns out that there is a remarkably simple and elegant way to handle the issue by maintaining 
what are known as **function tables** (called [virtual function tables](https://en.wikipedia.org/wiki/Virtual_method_table) in OOP jargon) at run time.

**The key to the solution to the problem is that the variable x must carry the information about
which class is it referring to at run time, and this information must
be used to dereference the function call at run time.**

The implementation details are outlined below:
1. **For each class, we will maintain a table of size 8 in the memory, listing labels of the
    methods of the class.** (Recall that we permit a class to have at most 8 methods). **The labels
    may be listed in the order in which the methods are defined in the class.** This table is called
    the **virtual function table** for the class. Since all the class definitions are known at compile
    time, the compiler can generate initial code to set up all the class tables in the stack region of
    the program. (Typically, at the beginning of the stack, allocating space before global variables.)

2. **The label for a method in the virtual function table will be set to the method's label in the
    corresponding class table entry.** Consequently, the label of a method in a descendent class will
    be the label of the method in the parent class if the method is not over-ridden. Referring to the
    example above, the label for the method fun() in the virtual function tables of classes A and B
    will be L1, whereas, the label will be L2 in the virtual function table of class C.

3. **The position of the label of a method in the virtual function tables of the parent class and all
    the descendent classes in a class hierarchy must be the same.** Referring to the above example,
    if the label of the method fun() is listed third in the virtual function table of A, then it must be
    listed third in the virtual function tables of classes B and C. Thus the index of a method's
    entry in the virtual function table will be the same for every class in a class hierarchy.
    The advantage of this convention is that while translating, **the position of a method's label
    relative to the base address of the virtual function table is completely determined at compile time.
    ** Note that since the language does not permit function overloading, the entry for a
    given method name in a virtual function table will be unique.

4. In view of the above, to translate _x.fun()_, all that needs to be determined at run time is
    the base address of the correct virtual function table. To keep track of this information, **the
    compiler must allocate two memory locations for each class variable - One for the usual pointer to
    the memory block allocated in the heap for storing the object's member fields .
    The second, to store a pointer to the virtual function table of the class to which
    the current object stored in the variable belongs to.**

Conceptually, each class variable holds a pair `[MemberFieldPointer, VirtualFunctionTablePointer]`.
The statement `x=new(B);` in the example above should do the following:

*   Allocate a block of 8 words in the heap and store the start address in the _MemberFieldPointer_ field of x.

*   set the _VirtualFunctionTablePointer_ field of x to the base address of the virtual function table of class B.

An assignment of the form y=x, if valid semantically, will result in both the pointers of x being
copied into the corresponding pointers of y.

Note that once the base of the correct virtual function table is known, invoking the right function
simply involves adding the offset of the function to the base and making a call to the
address (label) stored in the virtual function table entry.

Note that the labels will be automatically translated to actual addresses during the
[label translation phase](../label-translation.md).

To implement virtual function tables on the eXpOS ABI, the suggested method is to allocate space
(8 words each) for storing the virtual function tables of each class in the stack before
allocating space for global variables.

**Read the [OExpL run time data structures documentation](../oexpl-run-data-structures.md)
before proceeding further.**

**Task 2:** Complete the OExpL compiler.

## Test Programs

Check your implementation with the following test cases :

- [Test Program 1: Testing the runtime binding of the variables of a class](../oexpl-testprograms.md#test4)
- [Test Program 2: Testing the correct set up of the virtual function table](../oexpl-testprograms.md#test5)
- [Test Program 3: Testing the implementation of inheritance and subtype polymorphism](../oexpl-testprograms.md#test6)
- [Test Program 4: Testing the implementation of inheritance and subtype polymorphism](../oexpl-testprograms.md#test7)

!!! question "Exercise 1"
    This exercise asks you to add a limited form of **function overloading** support to the language.
    Here, when a descendent class overrides a method of a parent class, it can re-define the function with
    a signature that is possibly different from that of the parent class. In this case, both the definitions
    will be active in the child class; with the compiler translating the call to the correct address (label)
    looking at the arguments. Make necessary modifications to the language syntax to support this form of function overloading.

!!! note
    Overloading and subtype polymorphism are two polymorphism types typiclly supported by most
    object oriented langauges. A third important type of polymorphism called
    _[parametric polymorphism](https://en.wikipedia.org/wiki/Parametric_polymorphism)_
    (templates in C++) has not been touched upon in this project.
