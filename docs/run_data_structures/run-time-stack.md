---
title: 'Run Time Stack Allocation'
hide:
    - toc
---

# Run Time Stack Allocation

## Introduction

During run-time, when an ExpL function is invoked, space has to be allocated for storing

- the arguments to the function,
- return value of the function,
- local variables declared in the function.

For this, an **activation record** is created in the stack for each function call (and the stack grows). The activation record is also used to save the state (registers in use) of the invoking function and some control information (like the return address of the calling program).

Each activation record has a base, and the **base pointer (BP)** is a machine register that points to the base of the activation record of the currently executing function. When one function invokes another, the base pointer value of the caller is pushed on to the stack and BP is set to point to the new activation record base. Upon return, the activation record is popped off the stack and old value of base pointer is restored. The **stack pointer (SP)** must always point to the top of the stack.

The **calling convension** fixes in what order arguments to a function must be pushed by the caller to the called function, the place in the activation record where the return value is expected to be written by the callee etc. The structure of the activation record explained below will clarify the calling convension.

The stack is assumed to grow downwards.

![](../img/data_structure_34.png)

When a function is invoked, a part of the activation record is set up by the caller and the rest is set up after invocation of the function. Similarly, when a function returns, the callee and the caller are responsible for removing the parts they have set up.

The following sequence of actions occur when a function A calls another function B.

1\. A pushes its machine state (registers in use) into the stack so that the registers are free for use in B.

2\. A pushes the arguments to B in the order they appear in the declaration.

3\. A pushes one empty space in the stack for B to place its return value.

4\. A invokes B. (This results in generation of a CALL instruction which results in pushing the instruction pointer into the stack and transfer of control to B).

Inside B, the following space allocations take place:

5\.  B saves the BP value of A to the stack and sets BP to the top of the stack.

6\.  B allocates space for local variables (in the order in which they appear in the delcaration).

This completes the activation record for B. If B later calls another function C, then it starts saving its registers, pushes arguments to C and so on.

When B completes execution the following sequence of actions take place:

1\.  B computes the return value and stores it in the space allocated for it in the stack.

2\.  B pops out the local variables.

3\.  The old BP value is popped off and saved into BP.

4\.  B returns (this results in generation of a RET instruction which results in setting the instruction pointer to the value saved in the stack).

On re-entry, A does the following:

5\.  Retrieve the return value from stack and save it to a new register. This is the result of the function call.

6\.  Pop off the arguments.

7\.  Restore the saved register context.

## Illustration

Consider the following example:
```
decl
    int result,factorial(int n);
enddecl
int factorial(int n){
    decl
        int f;
    enddecl
    begin
        if( n==1 || n==0 ) then
            f = 1;
        else
            f = n * factorial(n-1);
        endif;
        return f;
    end
}
int main(){
    decl
        int a;
    enddecl
    begin
        read(a);
        result = factorial(a);
        write(result);
        return 1;
    end
}
```

1.  The global variables are allocated statically in the initial portion of the stack. Note that stack begins at `4096` according to the [ABI specification](../abi.md).

    ![](../img/data_structure_39.png)

2.  Assuming that user inputs 3 resulting in `a=3`, the main functions sets up stack locations for its local variables and calls the function `factorial(3)` after setting up a part of the callee's activation record.

    ![](../img/data_structure_40.png)

3.  `factorial(3)` saves the old Base pointer and sets up locations for its local variables.

    ![](../img/data_structure_41.png)

4.  `factorial(3)` calls `factorial(2)` and the activation record of `factorial(2)` is setup similar to the above steps.

    The register `R0` is assumed to be used for temporary storage of the value of n in the expression `n * factorial(n-1)` i.e, 3.

    ![](../img/data_structure_42.png)

5.  Activation record for `factorial(1)` (called by `factorial(2)`) is setup similarly.

    ![](../img/data_structure_43.png)

6.  `factorial(1)` calculates the result and returns it by setting the value at return value location and pops off it local variables and sets back the base pointer.

    ![](../img/data_structure_44.png)

7.  Similarly, `factorial(2)` calculates the steps and pops off its activation record till the result value after setting back the old base pointer.

    ![](../img/data_structure_45.png)

8.  Similarly, `factorial(3)` also calculates the result and returns it to the main function.

    ![](../img/data_structure_46.png)

9.  Main function calculates and sets the `result` variable.

    ![](../img/data_structure_47.png)

