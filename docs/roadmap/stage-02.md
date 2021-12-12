---
title: 'Stage 2. Introduction to static storage allocation'
hide:
    - toc
---

# Stage 2. Introduction to static storage allocation



!!! quote "Time estimate"
    0.5 week, 5-10 hours/week

!!! abstract "Prerequisites"
    1. You must be comfortable with LEX and YACC. (If you are not, you must first do
        [LEX tutorial](../lex.md), [YACC Tutorial](../yacc.md) and [YACC+LEX tutorial](../ywl.md).)
    2. You must have completed the [XSM environment tutorial](../xsm-environment-tut.md)
        before starting this stage.

!!! example "Learning Objectives"
    In this stage, you will extend the expression evaluator of the previous stage to support a
    set of pre-defined variables with Input/Output and assignment statements. You will get
    introduced to the notion of **static storage allocation** enroute. You will also learn to
    differentiate between statements and expressions and also construct an
    _abstract syntax tree representation_ for a program.


Consider a simple programming language with the following syntax:


```c
Program ::= BEGIN Slist END | BEGIN END

Slist ::= Slist Stmt | Stmt

Stmt ::= InputStmt | OuptputStmt | AsgStmt

InputStmt ::= READ(ID);

OutputStmt ::== WRITE(E);

AsgStmt ::== ID = E;

```

Apart from the [literal tokens](../lex.md), BEGIN, END, READ, and WRITE are tokens corresponding to
keywords "begin", "end", "read" and "write". ID is a token for variables. We will permit only
variables `[a-z]` in this stage.

To support variables to appear in expressions, you must add the rule `E := ID` to the expression
syntax used in Stage 1. The above syntax defines a small programming language that permits just
straight line programs (programs without conditionals, loops, jumps or such control transfer
constructs).
There are only 26 pre-defined variables that are supported – a, b,c,..,z. A typical program would look like:

```
begin
    read (a);
    read (b);
    d = a + 2 * b;
    write (a+d);
end;
```

We will assume that variables can store only integers. Handling variables of multiple
types will be taken up in subsequent stages.

A conceptual point to note here is that apart from the addition of variables, the extended language
now has two kinds of constructs – expressions and **statements**. While an expression evaluates to
a value (in this case, we limit ourselves to integer expressions), a statement commands the execution
of some action by the machine. For example, the statement `read(a);` instructs the action of
reading a variable from the console into a variable a. `write(a+d);` instructs evaluation of
the expression `(a+d)` and printing the result into the console output.

Another important conceptual point to note is that the introduction of variables also demand
binding them to storage (memory) locations. The storage location associated with a variable must
hold the value of the variable at each point of program execution. A statement (like the assignment
statement or a read statement) that alters the value of a variable must result
in a change the value stored in the corresponding storage location.

In the present case, the compiler can fix the address for each variable in memory right at the
time of program compilation. Since the ABI stipulate that storage allocation must be done in the
stack region, we can pre-allocate the first 26 memory locations in the stack region of
[memory](../abi.md#nav-virtual-address-space-model) for the variables a-z. Thus, variable `a` will
refer to contents of address 4096, `b` to contents of address 4097 and so on. Any time the compiler
encounters the variable – say `a`, the address to be looked at is fixed – in this case `4096`.
Such allocation policy is called **static allocation**. In later stages you will encounter
situations where it will not be possible for the compiler to fix memory address of a variable
at compile time. This leads to **run time** and **dynamic memory allocation** policies.
For now, we will be content with static allocation.

To implement the above, your compiler must:

1. Fix the storage location for each variable. As noted above, the first 26 locations of the
    stack region starting at address 4096 may be assigned for a to z. Note that the XSM machine
    can store an integer in a single memory location. Hence, for each variable we need to allocate
    only 1 memory word. Note that "allocation" here means that while generating code, the compiler
    assumes that the variable `a` is stored in location `4096`, b in location `4097` and so forth.

    !!! note
        Some programming languages stipulate that variables must be initialized to zero.
        In that case, the compiler must generate code to `MOV 0` to each of these locations before
        generating code for statements in the program. Some machines provide machine instructions
        that support initializing memory to zero. Certain operating systems would have initialized
        all memory regions (except those to which code is loaded into) to zero at load time.
        We will not pursue these issues here.

2. To translate an assignment statement, the compiler must generate code to evaluate the expression
    and then MOV the contents of the register storing the result to the memory location allocated for the variable.
3. To translate a Read statement, the compiler must generate code to invoke the
    [library](../abi.md#nav-eXpOS-system-library-interface) function for read, passing the address
    of the variable as argument. Write is implemented similarly.

But before getting into code generation, we must create an **abstract syntax tree (AST)** for the
program. An abstract syntax tree is a tree representation of the program, just like an expression
tree for expressions. An abstract syntax tree for the above program would look like the following:

![](../img/ast_stage2.png)

Observe that each node now needs to store distinguishing information like:

1. Whether it corresponds to a variable, constant, operator, assignment statement, write statement
    or read statement.
2. In case of operators, the information on the operator must be present. In the case of constants,
    the value must be stored in the node. In the case of variables, the node must contain the variable name.
3. There are also connector nodes which simply join two subtrees of statements together.

This leads to the definition of the following node structure:

```c
typedef struct tnode {
	int val;	    // value of a number for NUM nodes.
	int type;	    // type of variable
	char* varname;	// name of a variable for ID nodes
	int nodetype;   // information about non-leaf nodes - read/write/connector/+/* etc.
	struct tnode *left,*right;	// left and right branches
}tnode;

/*Create a node tnode*/
struct tnode* createTree(int val, int type, char* c, struct tnode *l, struct tnode *r);
```

!!! question "Task 1"
    Use Yacc and Lex to generate abstract syntax tree representation of a program.
    A file containing the source program will be input to your program.

Thus, after parsing, we use the syntax directed translation scheme of YACC to construct an
intermediate representation – namely, the abstract syntax tree. This phase of compilation is
sometimes called the **front end** of the compiler. The next step is to recursively traverse
the expression tree to generate executable code. This is typically called the **back end**.
The output of the front end is generally a **machine independent intermediate representation**
like the AST. The back end of course will be dependent on the target platform.


!!! question "Task 2"
    Modify `CodeGen()` function of Stage 1 to generate code for the abstract syntax tree generated as Task 1 above.

In the next stage, we will see how program control instructions like if-then-else can be incorporated into the language.


!!! note
    An abstract syntax tree is an [intermediate representation](https://en.wikipedia.org/wiki/Intermediate_representation)
    of the source program in a tree form suitable for code generation. There are several other forms of intermediate
    representations like the three address code form, the static single assignment form etc.
    This roadmap will be based on the abstract syntax tree representation.

In commercial strength compilers, the source is first translated to intermediate forms like the
three address form which is a **lower level representation** (that is the intermediate form is
closer to machine code) than the AST. Typically machine independent
[code optimizations](https://en.wikipedia.org/wiki/Optimizing_compiler) are performed on the
intermediate code and only then the back-end code generation is run. This step is followed by
another set of machine dependent code optimizations before the target file is finally generated.
As these issues are beyond the scope of our project, we will not dwell
into these matters further in this roadmap.

!!! question "Exercise 1"
    Build an **evaluator** for the program. (Hint: Your front end does not change. But, instead of
    generating code from the AST, you can recursively "evaluate" it. For storage allocation of variables,
    you can simply declare an array that can store 26 integers and allocate one entry for each variable).

!!! note
    The compiler generates target code which must be executed by the target machine. In our case, 
    the compiler you wrote as Task 2 actually is a **cross compiler**. This means that your compiler
    generated target code that is not for your host system, but on some other target platform – which
    in our case the simulated XSM machine. The evaluator done in Exercise 1 actually does not generate
    "code" for any machine. Instead, it executes the program in "then and there".
    Such a program could be classified as an **interpreter**. (Unfortunately, the standard
    terminology in literature associated with the term "interpreter" seems to be contradictory to this classification).

