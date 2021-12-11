---
title: 'OExpL Specification'
---
# OExpL Specification

## Introduction

An informal specification for a minimal object oriented extension of the ExpL languge with support for [Inheritance](https://en.wikipedia.org/wiki/Inheritance_(object-oriented_programming)) and [subtype polymorphism](https://en.wikipedia.org/wiki/Polymorphism_(computer_science)) is outlined here.

## Class Definitions

Classes extend the notion of ExpL types. A class [encapsulates](https://en.wikipedia.org/wiki/Encapsulation_(computer_programming)) **_member fields_** and **_member functions_** (called **methods** in OOP jargon). The following is an example for a class definition. Standard syntax and semantics conventions followed in languages like Java and C++ are assumed.

All the class definitions in a program must be placed together, between the keywords _class_ and _endclass_. Declaration of class variables follow exactly the same syntax as declaration of user defined type variables in ExpL. **The member fields of a class must be declared before member functions are declared.** Class definitions must be placed after type definitions, but before global declarations. (Further example programs are provided at the end).

Since OExpL is designed for pedegogical purposes, the following restrictions are imposed to simplify implementation.

1. **The member fields of a class may be of type integer, string, user defined types, previously defined classes or of the same class.** Thus the language supports both [_Composition and Inheritance._](https://en.wikipedia.org/wiki/Composition_over_inheritance)
2. **Member fields of a class are private to the class** in the sense that they can be accessed only by the methods defined in the class. The language does not permit _access modifiers_ like [**public** or **protected**.](https://en.wikipedia.org/wiki/Access_modifiers)
3. **Class variables can be declared only globally. Class variables cannot be arguments to functions; a function cannot have a class as its return type** and class variables cannot occur as local variables within functions.
4. All the member fields and methods should be declared between _decl_ and _enddecl_.
5. In methods defined within a class, the special keyword **self** refers to the instance of the class through which the method was invoked. ( The usage is similar in spirit to [**this** in C++.](https://en.wikipedia.org/wiki/This_(computer_programming)) )
6. The methods defined in a class may have parameters as well as local variables. The syntax and semantics rules are similar to other ExpL functions. However, there are some important differences:

    a) **Methods** of a class, apart from its arguments and local variables, **have access only to the member fields of the corresponding class.**  
    b) **A method can invoke only functions of the same class or functions inherited from its parent class or methods of class variables occuring as member fields of the class.** (Be aware of the [_fragile base class problem_](https://en.wikipedia.org/wiki/Fragile_base_class)).

To put in short

1. class variables can only be global.
2. Member functions of a class can access only its member fields, methods, local variables, arguments and methods of member fields.
3. Member fields of a class can be accessed from outside only through member functions of the class.

## Inheritance

The language supports class **extension**. A class defined by extension of another class is called a _derived class_ (or _child class_) of the _parent class_ (sometimes called the _base class_). The following example demonstrates the syntax of class extension. The language does not support [multiple inheritance](https://en.wikipedia.org/wiki/Multiple_inheritance) .

The semantics of class extension can be summarized as follows:

1. **The derived class inherits all member fields of the parent class automatically.** If additional fields are defined, they will be specific to the derived class. **The derived class cannot re-declare a member field already declared in the parent class.**
2. **The derived class inherits only those methods of the parent class which are not re-defined (overridden).** If a method is overridden, the new definition will be the only valid definition for the derived class. All the overridden methods must be declared again in the derived class. **The signature of the overridden method must match exactly in both number and types of arguments with the signature of the function in the parent class.** Only one function of the same name is permitted in a class. Thus, the language does **not** permit [function overloading]( https://en.wikipedia.org/wiki/Function_overloading).

## Class variables and Instantiation

Class variables are declared just like other variables in the global declaration section after type definitions and class definitions.

Example:

Object instance is created for a variable of a class with the built-in function _new_. The language does not support [constructors and destructors](<https://en.wikipedia.org/wiki/Constructor_(object-oriented_programming>). Hence intitialization of objects has to be done explicitly. An object can be deallocated using the built-in function _delete_. The function _new_ will create an object of a specified class at run time, and assigns a _reference_ to the object into a variable. A variable of a given class may be assigned a reference to an object of any desendent class using _new_. Access semantics of class variables is similar to ExpL user-defined-types, except for the details associated with methods defined within classes. These details are described below.

## Subtype Polymorphism

**A variable of a parent class can refer to any object in its inheritance hierarchy.** That is, if class B extends class A and class C extends class B, then a variable of class A can refer to objects of classes A, B or C, whereas a variable of class B can refer to any object of class B or C. (References are set using the ExpL _assignment statement_ in the normal way, as with user defined types. The function _**new**_ can also be used to set the reference for a variable). When a method is invoked with a variable of class B or C, if the method is inherited from an ancestor class, the ancestor's method is invoked. This is illustrated by the following examples.

## Run time Binding

Suppose that a variable declared to be of a parent class in an inheritance hierarchy holds the reference to an instance of a derived class. On invocation of a method of the parent class variable, if the method is over-ridden by the derived class, the method of the derived class is invoked.

This pivotal feature of object oriented programming compilcates the compiler implementation because **at the time of generating code for a method invocation, the correct method to call may not be known,** as illustrated in the example below.

In the above code, the value of the variable n read from the input at run-time determines whether the variable _arbitrary_ refers to an object of the class - person or the class - student. Consequently, at compile time, we cannot decide whether the call _arbitrary.printDetails()_ must be translated to an invocation of the function in the parent class or the child class.

To resolve such function invocations, the method of [dynamic binding](https://en.wikipedia.org/wiki/Late_binding) must be implemented using the [The virtual function table method](https://en.wikipedia.org/wiki/Virtual_method_table) .A tutorial explaining the implementation of virtual function tables is given [here](http://silcnitc.github.io/oexpl-run-data-structures.html) .

**Important Note :** If a variable of a parent class holds the reference to an object of a descendent class, only methods defined in the parent class are allowed to be invoked using the variable of the parent class.

## Sample Programs

Example Programs are [here](oexpl-testprograms.html).

## Appendix

#### Keywords

The following are the reserved keywords in OExpL and it cannot be used as identifiers.

class

endclass

extends

new

delete

self
