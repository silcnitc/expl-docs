---
title: 'OExpL Compile Time Data Structures'
---

# Compile Time Data Structure for OExpL

## Class Table

OExpL compilation requires only one additional data structure - the class table. The class table stores information pertaining to all the classes declared in an OExpL program. For a class it stores member fields, member functions, name of the class and parent class pointer.

## Structure

The structure of Class Table(CT) is as follows:

```c
struct Classtable {
 	char *Name;                           //name of the class
	struct Fieldlist *Memberfield;        //pointer to Fieldlist
	struct Memberfunclist *Vfuncptr;      //pointer to Memberfunclist
	struct Classtable *Parentptr;         //pointer to the parent's class table
	int Class_index;                      //position of the class in the virtual function table
	int Fieldcount;                       //count of fields
  	int Methodcount;                      //count of methods
	struct Classtable *Next;              //pointer to next class table entry
};
```

!!! note
    Memberfield list is used to store the information regarding the type, name, fieldindex and type of class of all the member fields of that class.

```c
struct Fieldlist{
	char *Name;			//name of the field
	int Fieldindex;			//position of the field
	struct Typetable *Type;		//pointer to typetable
	struct Classtable *Ctype;	//pointer to the class containing the field
	struct Fieldlist *Next;		//pointer to next fieldlist entry
};
```

Memberfunc list is used to store the information regarding the type, name of the function, argument list, it's flabel and it's position.
```c
struct Memberfunclist {
 	char *Name;                      //name of the member function in the class
	struct Typetable *Type;          //pointer to typetable
	struct Paramstruct *paramlist;   //pointer to the head of the formal parameter list
	int Funcposition;                //position of the function in the class table
 	int Flabel;                      //A label for identifying the starting address of the function's code in the memory
	struct Memberfunclist *Next;     //pointer to next Memberfunclist entry
};
```

## Associated Methods

- `#!c struct Classtable* CInstall(char *name,char *parent_class_name)` :

    Creates a class table entry of given 'name' and extends the fields and the methods of parent class and returns a pointer to the newly created class entry.

- `#!c struct Classtable* CLookup(char *name)`:

    Search for a class table entry with the given 'name', if exists, return pointer to class table entry else return NULL.

- `#!c void Class_Finstall(struct Classtable *cptr, char *typename, char *name)` :

    Installs the field into the given class table entry which is given as an argument.

- `#!c void Class_Minstall**(struct Classtable *cptr, char *name, struct Typetable *type, struct Paramstruct *Paramlist)` :

    Installs the method into the given class table entry which is given as an argument.

- `#!c struct Memberfunclist* Class_Mlookup(struct Classtable* Ctype,char* Name)` :

    Search through the VFunclist of the class using Ctype that is being parsed and return pointer to the entry in the list with function name as _Name_. Returns NULL if entry is not found.

- `#!c struct Fieldlist* Class_Flookup(struct Classtable* Ctype,char* Name)` :

    Search through the Memberfield of the current class using Ctype that is being parsed and return pointer to the entry in the list with variable name as _Name_. Returns NULL if entry is not found.

## Illustration

Here is an example illustrating it.
```
class
Person{
	decl
		str name;
		int age;
		int printDetails();
		str findName();
		int createPerson(str name, int age);
	enddecl
	int printDetails(){
		decl
		enddecl
		begin
			write(self.name);
			write(self.age);
			return 1;
		end
	}
	str findName(){
		decl
		enddecl
		begin
			return self.name;
		end
	}
	int createPerson(str name, int age){
		decl
		enddecl
		begin
			self.name=name;
			self.age=age;
			return 1;
		end
	}
}     /*end of Person class */
Student extends Person{

	decl
		int rollnumber;               /*  The members name and age are inherited from the parent class */
		str dept;
		int printDetails();
		int createStudent(str name, int age,int rollNo, str dept);
	enddecl
  	int createStudent(str name, int age,int rollNo, str dept){
		decl
		enddecl
		begin
			self.name =name;
           		self.age = age;
            		self.rollnumber = rollNo;
            		self.dept = dept;
            		return 1;
		end
	}
	int printDetails(){  /* This function is also overridden in the derived class */
		decl
		enddecl
		begin
			write(self.name);
			write(self.age);
			write(self.rollnumber);
			write(self.dept);
			return 1;
		end
	}         /**  The derived class inherits the findName() function from the parent **/
}  /* end of student class */
endclass
```


1.  As soon as the compiler encounters the class name, it installs the class name and the parent class name if present into the class table. Subsequently, If there is an extension to the parent class, all the member fields and methods of parent class are inherited. Following is how class table looks when class _Person_ is installed.  
    [![](/img/class_table_1.png)](/img/class_table_1.png)
2.  Following is how class table looks when class _Student_ is installed.  
      
    [![](/img/class_table_2.png)](/img/class_table_2.png)

