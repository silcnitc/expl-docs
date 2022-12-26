---
title: 'Type Table'
hide:
    - toc
---

# TypeTable

## Introduction

The Type Table stores information regarding the various user defined types in the source program. The compiler creates an entry in the Type Table for each user defined type. In addition to this, there are default entries created for [primitive types](../expl.md#nav-data-types) (_int_, _str_, _boolean_) and special entries _null_ and _void_ for the internal purposes of the compiler. The default and special entries are made beforehand whereas entries for user defined types are made as the [Type Definition](../grammar-outline.md#TypeDefBlock) section of the input ExpL program is processed.

## Structure

The structure of Type Table is as follows:
```c
struct Typetable{
    char *name;                 //type name
    int size;                   //size of the type
    struct Fieldlist *fields;   //pointer to the head of fields list
    struct Typetable *next;     // pointer to the next type table entry
};
```

The variable 'fields' is a pointer to the head of 'fieldlist'. Here 'fieldlist' stores the information regarding the different fields of a user-defined type.
```c
struct Fieldlist{
  char *name;              //name of the field
  int fieldIndex;          //the position of the field in the field list
  struct Typetable *type;  //pointer to type table entry of the field's type
  struct Fieldlist *next;  //pointer to the next field
};
```

## Associated Methods

- `#!c void TypeTableCreate()`: Function to initialise the type table entries with primitive types _(int,str)_ and special entries_(boolean,null,void).
- `#!c struct Typetable* TLookup(char *name)` : Search through the type table and return pointer to type table entry of type 'name'. Returns NULL if entry is not found.
- `#!c struct Typetable* TInstall(char *name,int size, struct Fieldlist *fields)` : Creates a type table entry for the (user defined) type of 'name' with given 'fields' and returns the pointer to the type table entry. The field list must specify the field index, type and name of each field. TInstall returns NULL upon failure. This routine is invoked when the compiler encounters a [type definition](../grammar-outline.md#TypeDefBlock) in the source program.
- `#!c struct Fieldlist* FLookup(Typetable *type, char *name)`: Searches for a field of given 'name' in the 'fieldlist' of the given user-defined type and returns a pointer to the field entry. Returns NULL if the type does not have a field of the name.
- `#!c int GetSize(Typetable * type)` : Returns the amount of memory words required to store a variable of the given type.

## Illustration

Let us consider the following sample code:
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
//...Global Declarations..
//...Functions....
```

1. The type table is first created and initialised to contain the default entries for each of the primitive and internal datatypes.
    This is done through a call to the function TypeTableCreate() from main function before yyparse() is called to start parsing the code.
    After the execution of TypeTableCreate() , the type table will be as follows:

    ![](../img/data_structure_1.png)

    !!! note
        The size field has been left out in the figure below.


2. After entry is created for linked list, the type would appear as shown below.

    ![](../img/data_structure_2.png)

3. Similar actions are carried out for user-defined type _marklist_ as well.

    ![](../img/data_structure_3.png)

4. Once the type declaration section in the input ExpL program, the type table is fully created and will not be changed later.

