Local Symbol Table    

[![](../img/logo.png)](index.html)

*   [Home](../index.html)
*   [About](../about.html)
*   [Code](#)
*   [Roadmap](../roadmap.html)
*   [Documentation](../documentation.html)

* * *

Local Symbol Table

  
  

*   [Introduction](#nav-introduction)
*   [Structure](#nav-structure)
*   [Associated methods](#nav-associated-methods)

Introduction
------------

In addition to the global symbol table, the ExpL compiler maintains a separate local symbol table _for each function_ for storing information regarding the **arguments** and **local variables**. Each function has its own list of local variables. So each function has its own Local Symbol Table.

Structure
---------

Associated Methods
------------------

*   struct Lsymbol\* **LInstall(char \*name,struct Typetable \*type)** : Creates a local symbol table with given 'name' and 'type' and also sets its 'binding'.
*   struct Lsymbol\* **LLookup(char \*name)** : search the LST and if any entry with given 'name' is found ,return the entry,else returns NULL.

Memory is allocated for local variables of a function from a seperate memory area called the **stack**. Hence, the binding for a local variable is the relative address of the variable with respect to the base of the **activation record** for the function. A machine register called the **base pointer** points to the base of an activation record of a function. The binding is added to the base pointer to obtain the address of a variable in the stack.

*               [Github](https://github.com/silcnitc)
*   [![Creative Commons License](../img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)

[Go up](#navtop)

*   [Home](../index.html)
*   [About](../about.html)