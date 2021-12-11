---
title: 'OExpL Run Time Binding Tutorial'
hide:
    - toc
---

# OEXPL Run Time Binding Tutorial

Consider the OExpL code given below extending the classes [here](oexpl-run-data-structures.md#nav-illustration)
```
decl
    A obj ;
enddecl

int main() {
  decl
    int temp,n;
  enddecl
  begin
    temp= initialize();
    read(n);
    if(n < 0)then
        obj = new(A);
    else
        if(n == 0)then
            obj = new(B);
        else
            if(n > 0)then
                obj = new(C);
            endif;
       endif;
    endif;
   write(obj.f0());             /* Have a careful look at this call */
    return 1;
  end
}
```

In the above code, the value of n is read from the console and not known at the compile time.

Consider the call write(obj.f0()); in the above code.

if (n < 0), the method f0() of the class A is invoked at run time. If (n = 0), the method f0() in the class B is invoked at run time and If (n > 0), the method f0() in the class C is invoked at run time. So, Depending on the value of n, the variable obj must be bound to a different class. **Note that the value of n is known only when the program is run (at run time) and not known statically determined (i.e., not known at compile time)**. Even though the three methods which are overridden has the same name f0(), the call addresses must be different in each case. The information in the virtual function tables will be useful here.

We noted earlier that the variable _obj_ has two words of memory assigned to it at compile time - one **member field pointer** and one **virtual function table pointer**. At run time, the member field pointer of _obj_ is used to store a pointer to the space allocated in the heap as done for user defined type variables. **The virtual function table pointer is used to store the start address of the virtual function table of the class assigned to _obj_ at run time.**

For instance, the statement _obj = new(A);_ must result in two actions:

1. The member field pointer of _obj_ must be set to a newly allocated heap block.
2. The virtual function table pointer of _obj_ must be set to the start address of the virtual function table of class A.

Similarly, _obj = new(B);_ must result in setting the virtual function table pointer of obj to the base address of the virtual function table of class B. Consequently, the compiler can resolve the call _obj.f0();_ by generating code to:

1. Use the virtual function table pointer field of _obj_ to find the address of the virtual function table of the class.
2. Look up the virtual function table to find the address (label) of the function f0().
3. Call the function (of course, after setting up the call stack).

In more detail, when the compiler encounters the statement _obj=new(B);_ code is generated for the following:

1. Call the library routine for Alloc to allocate heap memory. The address returned is stored into the member field pointer of obj.
2. Set the virtual function table pointer of obj to the base address of the virtual function table of the class B (i.e, 4104).

Note that before generating code, compile time check whether B is a descendant class of the class of obj (in this case A) must be made.

For the call obj.f0(), the compiler must do the following:

1. Find the index of _f0()_ in the class table entry of _obj_. Here _obj_ is declared to be of class A and the index (look up Funcposition field in the member function list of class A) of _f0()_ in class A will be 0.
2. Generate the code to move to a register (say R0) the label of the function at index 0 from the virtual function table of _obj_. The virtual function table pointer field of obj will point to the virtual function table.
3. Generate code to push the "self object" into the stack as argument (i.e., the member field pointer of obj and the virtual function table pointer of obj must be pushed as arguments to the call to f0() - See function call convention for methods below .)
4. Generate code to call the function using the label obtained in step 2. (call R0).
5. Generate code to unwind the stack and extract return values upon return from the call.
Note that compile time check whether f0() is a method in the class A to which obj is declared to must be done before proceeding to code generation.

## Run time Stack Management for Method invocations

While generating the code for invoking a method of a class using an object of the class (for example, in the call obj.f0() above, f0() is invoked using the object obj of class A) **the member field pointer and virtual function table pointer of the object must be pushed to the stack in addition to normal arguments of the function.** We will follow the convention that these two values will be pushed before other arguments are pushed. This is how the runtime stack looks, when a method of a class is called.

[![](img/runtimestackoexpl.png)](img/runtimestackoexpl.png)
For instance, in the above example, if the value read from the input is 0, the following figure shows the run time stack.

[![](img/runtimestackoexpl2_1.png)](img/runtimestackoexpl2_1.png)

### Need For Pushing The Object

According to OExpL Specification, a member field or method should be accessed using self. In order to find out which object we are talking about,when we say self, we need to push the object into the stack.

Consider the following OExpL program,

```
class
A
{
  decl
      int i;
      int f0();
      int f1();
  enddecl

  int f0() {
    decl
        int c;
    enddecl
    begin
        c = self.f1();    // call to f1() from f0()
        write(self.i);
        return 1;
    end
  }
  int f1() {
      decl
      enddecl
      begin
          self.i=0;
          write("In A F1");        // This is printed when f1() is invoked from an object of class A
          return 1;
      end
  }
}
B extends A
{
  decl
     int f1();      // B overrides f1
  enddecl

  int f1() {
        decl
        enddecl
        begin
            self.i=1;
            write("In B F1");   // This is printed when f1() is invoked from an object of class B
            return 1;
        end
  }
}
endclass

decl
  int n;
  A obj;
enddecl

int main() {
  decl
  enddecl
  begin
      read(n);
      if(n>0) then
        obj = new(A);
      else
        obj = new(B);
       endif;
        n = obj.f0();     // f0() contains a call to f1(). f1() is overriden by class B
        return 1;
  end
}
```

For **obj1**, the member field **height** must be set to 10. For **obj2**, the member field **height** must be set to 20.

In the method definition for the method **setHeight**, the member field **height** is set by using the statement, **self.height = a;**.
By this statement alone, we cannot say which object is **self** referring to. To identify which object is **self** referring to, we push the calling object to the stack.

If **obj1** calls the method **setHeight**, the member field pointer and virtual function table pointer of **obj1** are pushed into the stack. Now, we know that **obj1** is calling the method (By looking at the runtime stack). So, we set the **member field height of obj1** by using the **member field pointer** and the **field position of member field height**. Member field pointer is the heap address given to the object for storing the member fields of the class it is assigned with.

The virtual function table pointer is used, when we invoke other methods using **self**. **p = self.setHeight(5);** //In the method defaultHeight().
For this we need to generate a CALL instruction. The call address is obtained from virtual function table.

