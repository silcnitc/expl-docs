OExpL Compile Time Data Structures    

[![](img/logo.png)](index.html)

*   [Home](index.html)
*   [About](about.html)
*   [Roadmap](roadmap.html)
*   [Documentation](documentation.html)

* * *

Compile Time Data Structure for OExpL

  
  

*   [Class Table](#nav-class-table)
*   [Structure](#nav-structure)
*   [Associated methods](#nav-associated-methods)
*   [Illustration](#nav-illustration)

Class Table
-----------

OExpL compilation requires only one additional data structure - the class table. The class table stores information pertaining to all the classes declared in an OExpL program. For a class it stores member fields, member functions, name of the class and parent class pointer.

Structure
---------

The structure of Class Table(CT) is as follows:

✛**NOTE**:

Memberfield list is used to store the information regarding the type, name, fieldindex and type of class of all the member fields of that class.

Memberfunc list is used to store the information regarding the type, name of the function, argument list, it's flabel and it's position.

Associated Methods
------------------

*   struct Classtable\* **CInstall**(char \*name,char \*parent\_class\_name) : Creates a class table entry of given 'name' and extends the fields and the methods of parent class and returns a pointer to the newly created class entry.
*   struct Classtable\* **CLookup**(char \*name) : Search for a class table entry with the given 'name', if exists, return pointer to class table entry else return NULL.
*   void **Class\_Finstall**(struct Classtable \*cptr, char \*typename, char \*name) : Installs the field into the given class table entry which is given as an argument.
*   void **Class\_Minstall**(struct Classtable \*cptr, char \*name, struct Typetable \*type, struct Paramstruct \*Paramlist) : Installs the method into the given class table entry which is given as an argument.
*   struct Memberfunclist\* **Class\_Mlookup** (struct Classtable\* Ctype,char\* Name) : Search through the VFunclist of the class using Ctype that is being parsed and return pointer to the entry in the list with function name as _Name_. Returns NULL if entry is not found.
*   struct Fieldlist\* **Class\_Flookup**(struct Classtable\* Ctype,char\* Name)) : Search through the Memberfield of the current class using Ctype that is being parsed and return pointer to the entry in the list with variable name as _Name_. Returns NULL if entry is not found.

Illustration
------------

Here is an example illustrating it.

1.  As soon as the compiler encounters the class name, it installs the class name and the parent class name if present into the class table. Subsequently, If there is an extension to the parent class, all the member fields and methods of parent class are inherited. Following is how class table looks when class _Person_ is installed.  
    [![](/img/class_table_1.png)](/img/class_table_1.png)
2.  Following is how class table looks when class _Student_ is installed.  
      
    [![](/img/class_table_2.png)](/img/class_table_2.png)

*   [Github](http://github.com/silcnitc)
*   [![Creative Commons License](img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)

Contributed By : [J.Ritesh](#)  
        [J.Phani Koushik](#)  
        [M.Jaya Prakash](#)

*   [Home](index.html)
*   [About](about.html)

  

window.jQuery || document.write('<script src="js/jquery-1.7.2.min.js"><\\/script>')