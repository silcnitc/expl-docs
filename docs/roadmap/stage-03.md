---
title: 'Stage 3: Adding Flow Control Statements'
hide:
    - toc
---

# Stage 3: Adding Flow Control statements

!!! quote "Time estimate"
    1 week, 5-10 hours/week

!!! abstract "Prerequisites"
    You must read the [label translation tutorial](../label-translation.md) before proceeding with this stage.

!!! example "Learning Objectives"
    In this stage, you will extend the straight-line-program compiler of Stage 2 to support control flow
    constructs like if-then-else, while-do, break and continue. You will encounter integer and boolean expressions
    and the notion of _type_ enroute. You will also learn the use of _labels_ for handling control flow constructs.

The if-then-else and the while-do constructs can be added to the source language of Stage 2 by adding the grammar rule:


```
Ifstmt  ::= IF (E) then Slist Else Slist ENDIF
        | IF (E) then Slist ENDIF;
Whilestmt ::= WHILE (E) DO Slist ENDWHILE;
```

To permit logical expressions, we need to add to the grammar the following productions:

```
E ::= E < E | E > E | E < =E | E >= E | E != E | E == E;
```

A simple program in this language to find the largest among three numbers would look like the following:

```
read(a);
read(b);
read(c);
if (a < b) then
    if (b < c) then Write(c); else Write(b); endif;
else
    if (a < c) then Write(c); else Write(a); endif;
endif;

```

Note that we continue to assume that variables hold only integer values. The first task in translation is to complete the front end.

There is one important conceptual point to understand here before proceeding to the front end implementation. With the introduction of logical expressions, there are two **types** of expressions in the language – **arithmetic expressions** and **logical expressions**. An arithmetic expression evaluates to an integer value whereas a logical expression evaluates to a **boolean value** – that is true/false.

The guard of an if-else statement or a while-do statement must be a boolean expression. On the other hand, the expression on the right side of an assignment statement must be of integer type as variables are assumed to hold integer values only. In other words, the statements given below are **invalid**.

```
if (a+b) then Write(c);
  OR
a = b < c;
```

Your compiler must flag a "_type mismatch_" error if such constructs are encountered during the AST construction process. A program with type errors must not pass the compiler's type check scrutiny and the compiler must report error without generating code. Type analysis is a part of the responsibilities of a compiler (normally classified under [semantic analysis](https://en.wikipedia.org/wiki/Semantic_analysis_(compilers))).

A simple way to handle this issue is to _annotate each node in the AST with a **type attribute**_ that indicates what is the type of the expression (or subexpression) with this node as the root.

For example, consider the AST for the following erratic expression.
```
d = ( a + b ) + ( c < 3 )
```
![](../img/ast3.png)

Here, the root of the AST is an assignment node which is **typeless**. (statements have no type, only expressions have a type associated with them). The left subtree of the root is a variable, and hence has type integer. The right subtree is a **+** node of type integer. Hence, at the root, there is no type mismatch. However, the right child of the right subtree has type boolean and does not match the operand type for the + operator. Hence the compiler must terminate compilation flagging error "type mismatch". Note that the compiler can stop processing when the first error is encountered without proceeding further with the tree construction.

To implement type checking, add a type field in the AST node structure.

```c
#define inttype 1
#define booltype 0
typedef struct tnode{
	int val;	    // value (for constants)
	int type;	    // type of the variable
	char* varname;	// variable name (for variable nodes)
	int nodetype;	// node type - asg/opertor/if/while etc.
	struct tnode *left,*right;	//left and right branches
}tnode;


/*Create a node tnode*/
struct tnode* createTree(int val, int type, char* varname, int nodetype, struct tnode *l, struct tnode *r);
```

At the leaf nodes of the tree, since you have either constants or variables, the type must be set to integer. Next, while constructing the tree for intermediate nodes, check whether the types of the children are compatible with the operator at the root. For instance, for the addition operation, the check could be as the following:

```
E :== E+E {
            if (($1->type != inttype) || ($2->type != inttype)) {
                error("type mismatch");
                exit();
            } else {
                $$->type = inttype);
            }
         }

```

If there is no mismatch, you must annotate root node `($$->type)` with the proper type (integer in the above case).

!!! note
    The above check is better done inside the `TreeCreate()` function so that the YACC file is not cluttered with C statements.

The essential idea is that the type of each node can by synthesized from the types of the subtrees. At any stage, the compiler may terminate flagging error if a type error is found.

!!! question "Task 1"
    Complete the front-end module (AST construction) for the programming language. You need to

    1. add additional lexical tokens for the new constructs
    2. make appropriate modifications in the tree node structure including provision for storing type attribute
    3. modify the TreeCreate() function to have three subtrees passed (for if-then-else) etc.

!!! question "Exercise 1"
    To test the implementation of Task 1, implement an **evaluator** for the expression tree.
    Test with simple programs like those for finding the largest of 3 numbers, sum of n numbers (n read from input) etc.

The next task is to complete the back-end code generation phase. For better clarity, we will split the task into two steps.

**Step 1:** Generate code with **labels**. At this stage labels will be placed at various control flow points of the target assembly code so that a JMP instruction will only indicate the label corresponding to the instruction to which transfer of program flow must happen.

**Step 2:** Replace the labels with addresses.

!!! note "Important note"
    You must have read the [label translation tutorial](../label-translation.md) before proceeding any further.

We will now look at Subtask 1. Consider the following statement:

```
while (a < b) {
  a = a + 1;
}
```

The expression tree for the above statement would look like:

![](../img/ast31.png)

Suppose variable a is bound to address 4096, b to address 4097, then our plan is to generate code that would look like the following:

```asm
L1:
MOV R0, [4096]  // transfer a to R0
MOV R1, [4097]  // transfer b to R1
LT R0, R1       // a<b
JZ R0,L2        // if (a<b) is false goto L2
MOV R0, [4096]  // transfer a to R0
ADD R0, 1       // add 1
MOV [4096], R0  // transfer sum back to a
JMP L1          // goto next iteration.
L2:
... Next Instruction ...

```

Note the use of labels L1 and L2 indicating control flow points in the above code.
A while statement involves two jumps and two labels. The labels are just symbols that are placed
at the start of instructions to which jump instructions must branch to. Placing labels in the code
relieves us from bothering about the exact memory address to which jump must be made.
Of course, this is only a temporary measure. The final target code must not contain labels.

Our strategy here is to first generate code with labels and then replace the labels with addresses.
To implement the plan, we may name labels in the program `L0`, `L1`,... We must design an `int GetLabel()`
function that returns the index of the next unused label. Thus the first call to `GetLabel()` returns `0`, next call returns `1` and so forth.

The code generation strategy for the while-do statement is illustrated by the following pseudo-code.

```c
int label_1 = getLabel();
int label_2 = getLabel();
fprintf (target_file "L%d", Label_1)        // Place the first label here.
Generate code for the guard expression.
Generate code to compare the result to zero and if so jump to label_2   // loop exit
Generate code for the body of the while loop.
fprintf(target_file, "JMP L%d", label_1);   // return to the beginning of the loop.
fprintf(target_file, "L%d", label_2);       // Place the second label here

```

!!! question "Task 2"
    Complete the code generation with labels for while-do, if-then and if-then-else constructs.

Now, we must complete Step 2 of replacing the labels with the correct addresses. This is explained in the [label translation documentation](../label-translation.md).

!!! question "Task 3"
    Read the link specified above and complete the label translation for if-then, if-then-else and the while-do statement.

!!! question "Exercise 2"
    Test your Task 3 code with the following programs:

    1. program to find the largest for a, b, c (values read from input)
    2. program to read numbers till 0 is input and output the sum.

!!! question "Task 4"
    Add **break** and **continue** statements. Code for these statements need be generated only
    if they appear inside some while loop. Otherwise, the compiler may simply ignore these statements,
    generating no code. (The primary task is to keep track of which label to jump to when one of these statements is encountered).

!!! question "Exercise"
    Add `repeat-until` and `do-while` statements to the language with standard semantics.

!!! note
    Often in practice, programming languages allow a program to be split into different functions, written in different source files.
    In such cases, each file is separately compiled and the compiler generates target code with labels without translating them into addresses.
    Even variable references will be symbolic and the actual addresses may not be determined. Such target files are
    called [object files](https://en.wikipedia.org/wiki/Object_file). The compiler will include symbol table information in
    the object file for translation later. A separate software called the [linker](https://en.wikipedia.org/wiki/Linker_(computing))
    will collect the information in all the symbol tables and combine the object files into a single executable file replacing labels
    and symbolic variable references with actual addresses.

