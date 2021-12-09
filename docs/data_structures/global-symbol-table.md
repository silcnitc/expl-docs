---
title: 'Global Symbol Table'
hide:
    - navigation
---

# Global Symbol Table

## Introduction
The global symbol table stores information pertaining to all the global variables and functions in an ExpL program.

## Structure

The structure of Global Symbol Table(GST) is as follows:

```c
struct Gsymbol{
    char *name;               //name of the variable or function
    struct Typetable *type;   //pointer to the Typetable entry of variable type/return type of the function
    struct Classtable *Ctype;  //THIS FIELD IS REQUIRED ONLY FOR OEXPL
    int size;                 //size of an array. (The sizes of all other variables in 1)
    int binding;              //stores memory address allocated to the variable
    struct Argstruct *arglist;//pointer to the head of argument list in case of functions
    struct ASTNode *fbinding; //pointer to the root of the abstract syntax tree of the function
                            //In case of compiler, the call address must be stored in fbinding
    struct Gsymbol *next;     //points to the next Global Symbol Table entry
};
```

!!! note
    `flabel` is a placeholder for the starting address of the function's code in memory. Any call to the function must translate into an assembly level call to this address. When the compiler makes the symbol table entry for a function during [function declaration](../grammar-outline.html#TypeDefBlock), the actual starting address is not determined, and instead a _pseudo address_ is assigned to flabel. A call to the function will be initially translated by the compiler to a call to this pseudo address. The actual memory address will be determined later (often in a separate pass) and the compiler will replace all pseudo labels with the correct addresses. Here, we assume that the compiler assigns for each function a unique integer in the _flabel_ field of the global symbol table as its function identifier or "pseudo-address". Thus, the pseudo-address can be used to identify the function during the [label translation phase](http://silcnitc.github.io/label-translation.html). When the compiler generates code, more human readable labels like F0, F1, F2, F3, ... are assigned to functions whose flabel field is set to pseudo-addresses 0, 1,2,3,...

`Paramlist` is used to store information regarding the types and names of the parameters. ParamStruct has the following structure.
```c
struct Paramstruct{
	char *name;	         //stores the name of the parameter
	struct Typetable *type;  //pointer to type table entry of parameter type
	struct Paramstruct *next;  //pointer to the next parameter
};
```

## Associated Methods

- `struct Gsymbol* GInstall(char *name,struct Typetable *type, int size, struct ParamStruct *paramlist)`: Creates a Global Symbol entry of given 'name', 'type', 'size' and 'parameter list' and assigns a static address('binding') to the variable (or label for the function).
- `struct Gsymbol* GLookup(char *name)` : Search for a GST entry with the given 'name', if exists, return pointer to GST entry else return NULL.

Illustration
------------

Continuing with earlier example, let us add Global declaration section to it.

```
type
	linkedlist{
		int data;
		linkedlist next;
	}
	marklist{
		str name;
		linkedlist marks;
	}
endtype
decl
	str studentname;
	int rollno,average,findaverage(linkedlist marks);
	marklist studentmarks;
enddecl
```

1.  As soon as the compiler encounters the global declaration of a variable or funtion, it is installed into the Global Symbol Table.
    Subsequently, the parameters are attached to the entry in the case of functions. Following is how GST looks when _studentname_ is installed.
    ![](../img/data_structure_5.png)
2.  Similarly for _rollno,average,findaverage(linkedlist marks)_, symbol table entries are formed and installed.  
    ![](../img/data_structure_6.png)]
3.  The final Global Symbol table looks as follows:  
    ![](../img/data_structure_7.png)

