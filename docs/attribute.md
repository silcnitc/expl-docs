---
title: 'ATTRIBUTE SYNTHESIS'
---
# ATTRIBUTE SYNTHESIS

## Introduction to attributes

In the [last section](yacc.html#navpassingvalues) of the YACC documentation we noted that it is possible to pass values associated with tokens from yylex() to yyparse(). We had described the term 'attribute' as a value associated with a token. YACC uses yylval to facilitate passing values from the lexical analyzer to the parser. In this document we will explore attribute values for non-terminals and how these attributes are handled by YACC internally. We will also be exploring the usage of YYSTYPE to define custom attribute types.

yylval is a global variable of the default return type [YYSTYPE](#navyystype) declared by YACC in y.tab.c. In the YACC documentation, we had seen an [example](yacc.html#navexy1) which illustrates the passing of attributes from yylex() to yyparse(). If the programmer were to use LEX to generate yylex(), then the attributes will have to be passed to yyparse() in a similar fashion i.e, using yylval (as shown below). In the LEX program, the yylex() simply returns the token by its name while the associated attribute is assigned to yylval which can be accessed by yyparse(). Note that, for LEX to return a token, the token must be declared in the declarations section of the YACC program. The following example is a LEX program which returns a token DIGIT when it finds a string matching the "number" pattern.

```c
%{

 #include "y.tab.h"
 #include<stdlib.h\>
 #include<stdio.h\>

%}

 number  \[0-9\]+

%%

 {number} {
                    yylval = atoi(yytext);
                    return DIGIT;
       }

 .  return \*yytext;

%%
```

In this example, we want to return the token DIGIT when an integer is found in the input stream. In addition to the token, we need to pass the value found in the input stream to yyparse(). The lexeme found in the input stream is a string which contains the integer found. [atoi()](http://en.cppreference.com/w/cpp/string/byte/atoi) is a built-in function of the type int defined in the stdlib.h header file. We use atoi() to obtain the integer equivalent of the lexeme found. The obtained integer value is then assigned to yylval.

The following code segment is a YACC program which declares the token DIGIT. Recall that the following program must be run with a [\-d flag](ywl.html#navdflag) to the yacc command. This flag is used to create the [y.tab.h](ywl.html#navdflag) file which facilitates a one way communication from YACC to LEX i.e., it is used to communicate the tokens declared in the YACC program to the LEX program. Let us refer to the task of LEX obtaining the token declarations from YACC as LEX _importing_ the declarations. The y.tab.h file contains declarations of all the tokens in the YACC program. Note that the LEX program above includes the y.tab.h file in the auxiliary declarations section to _import_ the declarations.

```c
%{
#include <stdio.h\>
%}

%token DIGIT

%%

start : expr '\\n'  {printf("\\nComplete");exit(1);}
    ;

expr:  expr '+' expr  {printf("+ ");}
    | expr '\*' expr  {printf("\* ");}
    | '(' expr ')'
    | DIGIT   {printf("%d ",$1);}
    ;

%%

yyerror()
{
    printf("Error");
}

main()
{
    yyparse();
    return 1;
}
```

## Non-terminal attributes

In addition to values associated with terminal symbols, YACC also allows values to be associated with non-terminals. We extend our notion of an attribute to: "An attribute is a value associated with a terminal or non-terminal grammar symbol". The attribute of a grammar symbol in a production can be referred to in the actions section of a rule using a special YACC syntax: $1 for the first symbol in the body of a production, $2 for the second symbol, $3 for the third and so on. For example consider the following example of a YACC rule.

```c
X: B C D
```

The value of a symbol 'B' can be referred to by $1, value of 'B' can be referred to by $2 and value of symbol 'C' can be referred to by $3. It is also possible to assign a value to the non-terminal which occurs as the head of a rule's production using $$. A value can be assigned to the symbol X using $$. Let's call these the _attribute stack variables_.

Consider the problem of displaying two numbers in an input stream if they occur as a pair separated by a comma. Also suppose that the numbers must be displayed ONLY after a pair is found. Let us look at a YACC program that solves the problem.

```c
pair: number ',' number  { printf("pair(%d,%d),$1,$3"); }
 ;
number: DIGIT   { $$=$1; }
 ;
```

In the program segment, the first rule displays the value of the 'number' symbols when found as a pair in the input stream. The action of the rule refers to the value of the first number symbol as $1 and and the second number as $3. (Note that $2 refers to the _literal token_ ',' which does not have any value associated with it). Since 'number' is a non-terminal, its attribute cannot be set by yylex(). [Recall](yacc.html#navprod) that every non-terminal symbol in the CFG must have a corresponding production with the non-terminal as the head. The attribute value of a non-terminal is set by the action of the rule which contains the corresponding production. In the example, the value of the non-terminal 'number' is set by the rule:

```c
number: DIGIT   { $$=$1; }
```

The action of the rule sets the value of 'number' (reffered to using $$) to the value of DIGIT (referred to using $1).

Sample I/O:

```
I: 2,5
O: pair(2,5)

I: 3,5,7
O: syntax error
```

## Attribute Stack

YACC maintains a parse stack to achieve [shift-reduce parsing](yacc.html#navshiftreduce). The parse stack contains grammar symbols (both terminal and non-terminal symbols) representing the current [configuration of the parser](yacc.html#). Similar to the parse stack, YACC also maintains an attribute stack to maintain the values of the grammar symbols in the parse stack. The attribute stack is _synchronous_ with the parse stack. _Synchronous_ because the i'th value on the attribute stack will be the attribute of the i'th symbol on the parse stack.

[Recall](lex.html#) that when a token has been identified by LEX in the input stream, it stores the token's value in yylval. When a token is [shifted](yacc.html#navparseact) to the parse stack, the value of yylval is pushed on to the attribute stack. Similarly, during a [reduction](yacc.html#navparseact) when a non-terminal replaces a production [handle](yacc.html#navparseact) on top of the stack, the attributes of the grammar symbols of the handle are popped from the stack and the attribute value of the non-terminal (head of the production) is pushed on to the attribute stack. The following example is a complete YACC program that evaluates an expression.

```c
%{
#include <stdio.h\>
int result;
%}

%token DIGIT

%left '+'
%left '\*'

%%

start : expr '\\n'  {result = $1; exit(1);}
    ;

expr:  expr '+' expr  {$$ = $1 + $2;}
    | expr '\*' expr  {$$ = $1 \* $3;}
    | '(' expr ')'   {$$ = $2;}
    | DIGIT   {$$ = $1;}
    ;

%%

yyerror()
{
    printf("Error");
}

main()
{
    yyparse();
    printf("Expression value = %d",result);
    return 1;
}
```

Sample I/O:

```
I: 2+3\*(4+5)
O: 29
```

Consider the two productions from the example.

```
expr:  expr '+' expr  {$$ = $1 + $2;}
expr:  DIGIT   {$$ = $1;}
```

When the input 2+3\*(4+5) is fed to the parser, yylex() reads the first character and finds a matching token DIGIT. yylex() returns DIGIT and asigns the value 2 to yylval. When yylex() has returned, yyparse() pushes DIGIT to the parse stack and the value of yylval to the attribute stack. $1, $2, $3 etc., refers to the attribute values on top of the stack before a reduction takes place. $$ refers to the attribute value of the non-terminal which is the head of the production which contains the handle. When the non-terminal is pushed on to the parse stack, the value of $$ is pushed on to the attribute stack. $$ refers to the symbol on top of the stack after a reduction has taken place. The following table shows the configuration of the parse stack and the attribute stack at every step of the parsing process. Assume that whenever yylex() returns a token with no attribute, yyparse() pushes a '**.**' to the attribute stack.

<table class="tg">
<tbody><tr>
<th class="tg-e3zv">PARSE STACK<br></th>
<th class="tg-e3zv">ATTRIBUTE STACK<br></th>
<th class="tg-e3zv">I/P BUFFER<br></th>
<th class="tg-e3zv">PARSER-ACTION EXECUTED<br></th>
</tr>
<tr>
<td class="tg-031e"></td>
<td class="tg-031e"></td>
<td class="tg-031e">2 + 3 *(4 + 5) $<br></td>
<td class="tg-031e">_</td>
</tr>
<tr>
<td class="tg-031e">DIGIT</td>
<td class="tg-031e">2</td>
<td class="tg-031e">+ 3* ( 4 + 5 ) $<br></td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr</td>
<td class="tg-031e">2</td>
<td class="tg-031e">+ 3 *( 4 + 5 ) $</td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr +<br></td>
<td class="tg-031e">2 <b>.</b></td>
<td class="tg-031e">3* ( 4 + 5 ) $</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + DIGIT<br></td>
<td class="tg-031e">2 <b>.</b> 3</td>
<td class="tg-031e">*( 4 + 5 ) $</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr<br></td>
<td class="tg-031e">2 <b>.</b> 3</td>
<td class="tg-031e">* ( 4 + 5) $<br></td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr + expr *<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>.</b></td>
<td class="tg-031e">( 4 + 5 ) $</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr* (<br></td>
<td class="tg-031e">2 <b>.</b> 3<b> . .</b></td>
<td class="tg-031e">4 + 5 ) $<br></td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr *( DIGIT<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 4</td>
<td class="tg-031e">+ 5 ) $</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr* ( expr<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 4 </td>
<td class="tg-031e">+ 5 ) $</td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr + expr *( expr +<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 4 <b>.</b></td>
<td class="tg-031e">5 ) $</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr* ( expr + DIGIT<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 4 <b>.</b> 5</td>
<td class="tg-031e">) $<br></td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr *( expr + expr <br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 4 <b>.</b> 5</td>
<td class="tg-031e">) $<br></td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr + expr* ( expr<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 9</td>
<td class="tg-031e">) $<br></td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr + expr *( expr )<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>. .</b> 9 .</td>
<td class="tg-031e">$</td>
<td class="tg-031e">SHIFT</td>
</tr>
<tr>
<td class="tg-031e">expr + expr* expr<br></td>
<td class="tg-031e">2 <b>.</b> 3 <b>.</b> 9</td>
<td class="tg-031e">$</td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr + expr <br></td>
<td class="tg-031e">2 <b>.</b> 27</td>
<td class="tg-031e">$</td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">expr</td>
<td class="tg-031e">29</td>
<td class="tg-031e">$</td>
<td class="tg-031e">REDUCE</td>
</tr>
<tr>
<td class="tg-031e">$expr</td>
<td class="tg-031e">29</td>
<td class="tg-031e">$</td>
<td class="tg-031e">ACCEPT</td>
</tr>
</tbody></table>

## YYSTYPE

Attributes of terminals can be passed from yylex() to yyparse() and attributes of a non-terminal can be **synthesized**. An attribute of a non-terminal grammar symbol is said to _synthesized_ if it has been calculated from the attribute values of it's children in the [parse tree](http://en.wikipedia.org/wiki/Parse_tree). A synthesized attribute is an attribute of a non-terminal than has been calculated using the attribute values of the symbols in the handle that it replaces. An example of synthesized attribute:

```c
Z: X   { printf("Result=%d",$1);}
X: A '+' B  { $$ = $1 + $3; }
```

The attribute value of X is a synthesized attribute as it has been calculated using the attribute values of the symbols in the handle that it replaces. The attribute stack consists of attributes of token and synthesized attributes. The attribute stack is of the type YYSTYPE i.e, all the [stack variables](#navstackvar) are of the type YYSTYPE. For example, in the above production, $$,$1 and $3 are all of the type YYSTYPE. YYSTYPE is a YACC-defined data type. yylval is declared by YACC to be of the type YYSTYPE. By default YACC defines YYSTYPE to be the type int. It means that, by default only integer valued attributes can be passed from yylex() to yyparse() and only integer attributes can be synthesized. If we were to attempt to assign any other value to yylval or any of the attribute stack variables, a type error would be flagged on compiling y.tab.c using gcc.

This default type definition of YYSTYPE can overriden with any built-in or userdefined data type. For example if we wanted to print the prefix form of an expression:

```c
expr: expr OP expr { printf("%c %c %c",$2,$1,$3);}
```

The type of YYSTYPE can be overriden manually as shown below. This line has to be added to the declarations section of the YACC program. This may be used (not recommended) to change the type of all the attributes from int to some other type.

```c
typedef char YYSTYPE;
```

But inorder to have multiple custom attribute values, YACC offers a useful feature called %union to customize the type of YYSTYPE. %union is useful when we require to have different tokens of different types. For example if we wanted some tokens to be of the type int and some tokens to be of the type char. The following code segment can be added to declarations section of the YACC program to achieve that.

```c
/* YACC Auxiliary declarations*/

/* YACC Declarations*/
%union
{
 char character;
 int integer;
};

%token <character> OP
%token <integer> NUMBER

%%
expr: expr OP expr { printf("%c %d %d",$2,$1,$3);}
  | DIGIT {$$=$1;}
  ;
%%
/* Auxiliary functions */
```

Note that the type of the attribute of each token must be mentioned when the token is being declared using the following syntax.

```c
%token <type> tokenname
```

`type` must be declared under %union prior to use in the declaration of a token. If the type of a token is not explicitly mentioned, no attribute value can be assigned to the token i.e, it is assumed to be of type [void](http://en.wikipedia.org/wiki/Void_type).

## References

[http://epaperpress.com/lexandyacc/pry1.html](http://epaperpress.com/lexandyacc/pry1.html)  
[http://www.sci.brooklyn.cuny.edu/~zhou/teaching/cis707/attr/tsld003.htm](http://www.sci.brooklyn.cuny.edu/~zhou/teaching/cis707/attr/tsld003.htm)  
[http://cs.nyu.edu/~gottlieb/courses/2008-09-fall/compilers/lectures/lecture-08.html](http://cs.nyu.edu/~gottlieb/courses/2008-09-fall/compilers/lectures/lecture-08.html)  
[http://en.wikipedia.org/wiki/Compilers:\_Principles,\_Techniques,\_and\_Tools](http://en.wikipedia.org/wiki/Compilers:_Principles,_Techniques,_and_Tools)  

## References

For further details on the topics covered in this document, the reader may refer to the following :

1. Compilers : Principles,Techniques and Tools by Alfred V.Aho, Monica S. Lam, Ravi Sethi and Jeffrey D.Ulman .
2. Modern Compiler Implementation in C by Andrew W.Appel
3. Flex & Bison by John Levine
4. [http://dinosaur.compilertools.net/](http://dinosaur.compilertools.net/)
