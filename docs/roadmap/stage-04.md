---
title: 'Stage 4: User Defined Variables and arrays'
hide:
    - toc
---

# Stage 4: User Defined Variables and arrays

!!! quote "Time estimate"
    1 week, 5-10 hours/week

!!! example "Learning Objectives"
    You will extend the language of Stage 3 to permit users to declare and use variables of _integer_ and _string_ types. You will learn **symbol table** management enroute.


In this stage, we allow the program to contain variable declarations of the following syntax:

```
Declarations ::= DECL DeclList ENDDECL | DECL ENDDECL

DeclList ::= DeclList Decl | Decl

Decl ::= Type VarList ;

Type ::= INT | STR

VarList ::= Varlist , ID | ID
```

We will assume hereafter that all variables used in a program must be **declared** in the declaration section of the program (between the _decl_ and _enddecl_ keywords). Since string type variables are allowed, we will allow string constants as well. (See [ExpL specification](expl.html#nav-constant) for details).

A simple program in this language to find the sum of numbers entered from the console (until a zero is entered) would look like the following:

```
decl
  int num, sum;
  str mesg;
enddecl

read(num);
sum = 0;
while (num != 0) do
  sum = sum + num;
  read(num);
endwhile;
write("sum is:");
write(sum);
mesg = "good bye";
write(mesg);
```

It is the responsibility of the compiler to track for various **semantic errors** as:

1. Flag error if any variable not declared is used.
2. Flag error if a type mismatch involving any variable is found.

To this end, while parsing declarations, the compiler transfers the information about variables in a compile time data structure called the **symbol table**. The symbol table stores the following information about each variable:

1. Name of the variable (known at the time of declaration).
2. Type (For the present stage, only integer/string).
3. Size (For the time being, we will assume that all variables have size one).
4. The memory **binding** of each variable – that is, static memory address determined by the compiler for the variable.

The first three entries are determined by the declaration of the variable. For the fourth, a simple strategy would be to allocate the first address (4096) for the variable declared first, 4097 for the next variable and so on. Note that here too we are fixing the address of each variable at compile time (**static allocation**).

The following structure may be used for a symbol table entry:
```c
struct Gsymbol {
  char* name;       // name of the variable
  int type;         // type of the variable
  int size;         // size of the type of the variable
  int binding;      // stores the static memory address allocated to the variable
  struct Gsymbol *next;
}

```

The symbol table entries for the program above would look as below:
![](https://silcnitc.github.io/img/gsymboltable1.png)

To implement the symbol table, you must write two functions. For a simple implementation, a linear linked list suffices. In modern compilers, hash tables are maintained to make search efficient.

```c
struct Gsymbol *Lookup(char * name);            // Returns a pointer to the symbol table entry for the variable, returns NULL otherwise.
void Install(char *name, int type, int size);   // Creates a symbol table entry.
```


!!! note
    You must check before installing a variable whether the variable is already present.
    If a variable is declared multiple times, the compiler must stop the compilation and flag error.

!!! question "Task 1"
    Complete the program to parse declarations and set up the symbol table entries and print out the contents of the symbol table.

The next task is to make necessary modifications to the AST construction and code generation. These are straightforward. Add a an additional field to the tree node structure

```c
typedef struct tnode{
    int val;                    // value of the constant
    char* varname;              // name of the variable
    int type;                   // type of the variable
    int nodetype;               // node type information
    struct Gsymbol *Gentry;     // pointer to GST entry for global variables and functions
    struct tnode *left,*right;  // left and right branches
}tnode;

```

While constructing the tree, if a variable is encountered, keep a pointer to the corresponding symbol table entry. Set the type field as well to the correct type. The rest of the type checking steps are exactly as in the previous stage. The AST of while loop present in the above code is as follows (the relevant part of the code is shown below for easy reference):
```
.
.
.
while (num != 0) do
    sum = sum + num;
    read(num);
endwhile;
.
.
.
```

![](https://silcnitc.github.io/img/ast_stage4.png)

There is no serious change to the code generation process, except that for variables, the binding address is obtained from the symbol table.

!!! note "Important Note"
    The XSM architecture is unrealistic in that allows a memory word to hold a string.
    Normally, in a real system, a string would require storing the characters one after another in
    consecutive memory locations as in an array. You will anyway learn array allocation immediately.

!!! question "Task 2"
    Complete the AST construction and code generation steps.

## Adding arrays

The next step is to allow declaration of arrays like:
```
decl
    ---
    int a[100];
    str names[20];
    ---
enddecl

```

The declaration syntax must permit:

```
Varlist ::= Varlist , ID[NUM] | ID[NUM]
```

To implement this, for each variable, you must reserve as much static space as specified by the declaration and set the size field in the symbol table to indicate the number of words allocated. The next variable must be allocated space only below this space allocated.

For instance, for the declaration,

```
decl
  int a[10], b;
enddecl

```

The binding field in the symbol table for the variable a may be set to address 4096. The size entry set to 10. This means that we are allocating locations 4096-4105 for the array. The next variable, b can be bound to the address 4106.

![](https://silcnitc.github.io/img/gsymboltable2.png)


!!! question "Task 2"
    Complete the implementation of single dimensional arrays.

!!! question "Exercise 1"
    Permit two dimensional arrays like:
    ```
        int a[10][10];
    ```
    Test your implementation with a program for multiplying two `n x n` matrices.

!!! question "Exercise 2"
    Permit `pointer type` variables as in the following declaration as in the C programming language.
    ```
    decl
      int x, *p;
      str p, *q;
    enddecl
    ```
    If you permit assignments like `p=&x;` and `q=&p;` , pointer variables may also be permitted in expressions like `*p=*q+1;`
    for referring to the data pointed to, as permitted in the C programming language.

    Semantic rules as in the C programming language may be assumed.

!!! note
    Right now, you are not equipped to do dynamic memory allocation for pointer variables (as done by the `malloc()` function of C).
    Hence, a pointer type variable can be used as a pointer to another statically declared variable of the corresponding type.
    Dynamic memory allocation will be discussed in later stages.


## Test Programs

Check your implementation with the following test cases :

1. **Bubblesort (iterative)**

    This test program reads elements into an array and sorts them using the classic bubblesort algorithm. (iterative version)

    **Input** : 1. Number of elements to be sorted from standard input. 2. Elements to be sorted

    **Output** : A sorted array of elements.

    To get the code for this test program [click here](testprograms/stage4/bubblesort.html).

2. **Nth Fibonacci Number(iterative)**

    This test program prints the nth fibonacci number

    **Input** : 1.An integer n
    **Output** : nth fibonacci number

    To get the code for this test program [click here](testprograms/stage4/fibaofn.html).

3. **Is Prime or Not**

    This program tests if a given integer is prime or not.

    **Input** : 1.An integer n
    **Output** : Prime if n is prime else not a prime.

    To get the code for this test program [click here](testprograms/stage4/prime.html).

4. **Sum of n factorials (iterative)**

    This program prints the sum to n factorial for a given n.

    **Input** : 1.An integer n
    **Output** : sum of factorial of all integers 1 to n.

    To get the code for this test program [click here](testprograms/stage4/sum-to-n-fact.html).
