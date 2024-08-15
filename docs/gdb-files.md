---
title: "GDB Input Files"
---

## lex.l file

```c
%{
  #include  "y.tab.h"
  #include "tree.h"
  void yyerror(char *);

%}


alpha   [a-zA-Z]

%%

{alpha}+  {yylval.p = createTree(yytext, NULL, NULL); return ID;}
"+"       {return PLUS;}
"-"       {return MINUS;}
"*"       {return MUL;}
"/"       {return DIV;}
[()]      {return *yytext;}
[ \t\n]           {}
.               {yyerror("invalid character\n");}


%%

int yywrap(void) {
  return 1;
}

```

## parser.y file

```c
%{
  //#define YYSTYPE tnode*
  #include <stdio.h>
  #include <stdlib.h>
  #include "tree.h"
  #include "tree.c"
  extern struct tnode* idptr;
  int yylex(void);
  extern FILE *yyin;
  extern char* yytext;
%}

%union{
  struct tnode* p;
}

%token <p> ID

%type <p> expr

%left PLUS MINUS
%left MUL DIV
%%


Program :  expr    {infixtoprefix($1);printf("\n");free($1);}
        ;

expr :  expr PLUS expr  {$$ = createTree("+",$1, $3);}
     | expr MINUS expr  {$$ = createTree("-",$1, $3);}
     | expr MUL expr  {$$ = createTree("*",$1, $3);}
     | expr DIV expr  {$$ = createTree("/",$1, $3);}
     | '(' expr ')'  {$$ = $2;}
     | ID      {$$=$1;}
     ;

%%

void yyerror(char const *s)
{
    printf("yyerror '%s' and '%s'",s,yytext);
}


int main(int argc, char*argv[]) {
  if (argc > 1) {
      FILE *fp = fopen(argv[1],"r");
      if (fp) {
        yyin = fp;
      }
  }
  yyparse();
  return 0;
}

```


## input file

```
abc+(bcd-efg)*hij
```

## tree.h file

This is the header file for tree.c file

```c
// node structure
typedef struct tnode {
  char *symbol;
  struct tnode *left, *right;
} tnode;

// prints the prefix expressions
void infixtoprefix(struct tnode* root);

/*Create  tnode*/
struct tnode* createTree(char *symbol, struct tnode* left, struct tnode* right);
```

## tree.c file

This file contains the helper functions for the yacc file, like the createTree(), infixtoprefix() etc.

The yacc file imports the tree.c file and tree.h file

```c
#include <string.h>

struct tnode* createTree(char *symbol, struct tnode* left, struct tnode* right) {
  struct tnode *k = (struct tnode*)malloc(sizeof(struct tnode));
  k->symbol = strdup(symbol);
  k->left = left;
  k->right = right;
  return k;

}

void infixtoprefix (struct tnode* root) {
  if (root != NULL) {
    printf("%s ", root->symbol);
    infixtoprefix(root->left);
    infixtoprefix(root->right);
  }
}
```
