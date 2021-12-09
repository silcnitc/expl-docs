---
title: 'Abstract Syntax Tree'
original_url: 'http://silcnitc.github.io/data_structures/abstract-syntax-tree.html'
hide:
    - navigation
---

# Abstract Syntax Tree

## Introduction

The machine independent **front-end** phase of a compiler constructs an intermediate representation of the source program called the **Abstract Syntax Tree (AST)**.This is followed by a machine dependent **back-end** that generates a target assembly language program.

## Structure

The node structure of Abstract Syntax Tree is as follows:
```c
struct ASTNode{
    struct Typetable *type;           //pointer to the type table entry
    int nodetype;                     //node type information,eg:NODETYPE_WHILE,NODETYPE_PLUS,NODETYPE_STMT etc
    char *name;                       //stores the variable/function name in case of variable/function nodes
    union Constant value;             //stores the value of the constant if the node corresponds to a constant
    struct ASTNode *arglist;          //pointer to the expression list given as arguments to a function call
    struct ASTNode *ptr1,*ptr2,*ptr3; //Subtrees of the node. (Maximum Subtrees for IF THEN ELSE)
    struct Gsymbol *Gentry;           //pointer to GST entry for global variables and functions
    struct Lsymbol *Lentry;           //pointer to the function's LST for local variables and arguements
};
```

The union Constant is used to store the value of an integer or sting constant.
```c
union Constant{
    int intval;
    char* strval;
};
```

## Associated Methods

### TreeCreate
```c
struct ASTNode* TreeCreate(
    struct Typetable *type,
    int nodetype,
    char *name,
    union Constant value,
    struct ASTNode *arglist,
    struct ASTNode *ptr1,
    struct ASTNode *ptr2,
    struct ASTNode *ptr3
)

```
Creates a node with the fields set according to the arguments passed. The function must be responsible for doing type checking, and will be successful only if the expression tree corresponds to an ExpL statement (or ExpL expression) with no syntax or semantic errors. The function returns a pointer to the root node of the AST upon success, NULL is returned otherwise. The arguments to the function includes pointers to the subtrees of the AST node (which must have been created earlier). Other arguments include the type of the ExpL expression represented by the AST node, a node type (described below) etc.
  
The _type_ field of an AST node is designed to contain a pointer to the typetable entry corresponding to the type of the expression represented by the AST node (The type is set to void for statements). The _type_ and _nodetype_ fields of an AST node may be assigned values according to the following convension. If an expression evaluates to NULL, its type must be set to void.

<div class="md-typeset__scrollwrap"><div class="md-typeset__table">
<table>
<tbody><tr>
<th>Nodetype</th>
<th>Type</th>
<th>Description</th>
</tr>
<tr>
<td>CONST</td>
<td>int/str</td>
<td>For interger and string constants. As a special case, if the constant is NULL, the type is set to <b>void</b>.</td>
</tr>
<tr>
<td>ID</td>
<td>int/str/<br>user-defined type</td>
<td>For all variable literals.</td>
</tr>
<tr>
<td>PLUS/MINUS/MUL/DIV</td>
<td>int</td>
<td>For arithmetic operators '+', '-', '*', '/', '%'. <br> 'ptr1' and 'ptr2' must be of type int and set to the AST's of the left and the right operands respectively.</td>
</tr>
<tr>
<td>GT/LT/GE/LE</td>
<td>boolean</td>
<td>For relational operators '&gt;', '&lt;', '&gt;=', '&lt;='. <br>'ptr1' and 'ptr2' must be of type int and set to the AST's of the left and the right operands respectively.</td>
</tr>
<tr>
<td>EQ/NE</td>
<td>boolean</td>
<td>For relational operator '==' or '!='. <br>'ptr1' and 'ptr2' must be set to AST of the left and the right operands respectively and both must be of the same type.</td>
</tr>
<tr>
<td>IF</td>
<td>void</td>
<td>For the conditional construct 'if'. <br>'ptr1' must be set to the AST of the logical expression and must be of type 'boolean', 'ptr2' must be set to the AST of list of statements corresponding to the 'then part' and 'ptr3' must be  set to the AST of list of statements corresponding to the 'else part'.</td>
</tr>
<tr>
<td>WHILE</td>
<td>void</td>
<td>For conditional construct 'while'. <br>'ptr1' is set to the conditional logical expression and 'ptr2' is set to AST of list of statements under the body of while construct.</td>
</tr>
<tr>
<td>READ</td>
<td>void</td>
<td>For input statement 'read', 'ptr1' must have nodetype ID or FIELD and type of 'ptr1' must be either 'int' or 'str'.</td>
</tr>
<tr>
<td>WRITE</td>
<td>void</td>
<td>For output statement 'write', 'ptr1' must be of type 'int' and 'str' and must be set to AST of the expression whose value is to be written to the standard output.</td>
</tr>
<tr>
<td>ASGN</td>
<td>void</td>
<td>For assignment statement (&lt;var&gt; = &lt;expr&gt;). <br> 'ptr1' must be set to an AST of nodetype ID or FIELD and 'ptr2' must be set to an AST of expression whose value will be assigned to lvalue given by 'ptr1'. The types of the variable and the expression must match.</td>
</tr>
<tr>
<td>SLIST</td>
<td>void</td>
<td>To form a tree with multiple statements. The execution semantics is that the sequence of statements represented by the left subtree 'ptr1' must be evaluated before evaluating 'ptr2'.</td>
</tr>
<tr>
<td>BODY</td>
<td>int/str/<br>user-defined type</td>
<td>For body of a function, type indicates the return type of the function. This is created when the definition of a function is processed.</td>
</tr>
<tr>
<td>RET</td>
<td>int/str/<br>user-defined type</td>
<td>For return statement of a function.</td>
</tr>
<tr>
<td>FUNCTION</td>
<td>int/str/<br>user-defined type</td>
<td>For function calls. The type must be same as the return type of the function. The field 'arglist' must be set to list of arguments for the function.</td>
</tr>
<tr>
<td>MAIN</td>
<td>int</td>
<td>For main function.</td>
</tr>
<tr>
<td>FIELD</td>
<td>int/str/<br>user-defined type</td>
<td>For user-defined types,( to handle expressions of the form <i>ID . ID</i>, <i>ID . ID . ID</i> , etc).</td>
</tr>


</tbody></table>
</div></div>
## Illustration

Consider the following program
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

1. Lets construct the abstract syntax tree step by step. The AST for conditional expression `n==1` (in line 9) will be as follows:
    ![](../img/data_structure_50.png)

2. Similarly we have AST fot `n==0` (in line 9) as follows.
    ![](../img/data_structure_51.png)

3. Next consider the complete conditional expression `n==1 || n==0`.
    ![](../img/data_structure_52.png)

4. Next we will form the AST for assignment statement `f = 1` (in line 10).
    ![](../img/data_structure_53.png)

5. Next, lets consider the statement `f = n * factorial(n-1)` which consists of arthimetic expressions with operands '-','\*' and an assignment statement.
    AST for `n-1` is as follows.
    ![](../img/data_structure_54.png)
    AST for `n * factorial(n-1)` is as follows.
    ![](../img/data_structure_55.png)
    AST for `f = n * factorial(n-1)` is as below.
    ![](../img/data_structure_56.png)

6. Following is the AST for the if condition.
    ![](../img/data_structure_57.png)

7. The AST for return statement is as folows
    ![](../img/data_structure_58.png)

8. Finally the AST for the factorial function will be as follows.
    ![](../img/data_structure_59.png)
