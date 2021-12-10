---
title: YACC
---

# YACC Examples

## YACC Program
in2post.y

```c
%{
    #include <stdio.h>
%}
%token DIGIT PLUS STAR
%%
start : expr '\\n'           {printf("\\nComplete");exit(1);}
    ;
expr:  expr PLUS expr        {printf("+ ");}
    | expr STAR expr     {printf("\* ");}
    | '(' expr ')'
    | DIGIT             {printf("%d ",$1);}
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

## LEX Program
in2post.l

```c
%{
    #include <stdio.h>
    #include "y.tab.h"
%}
%%
[0-9]+	return DIGIT;
'+'	return PLUS;
'*'	return STAR;
'('	return \*yytext;
')'	return \*yytext;
%%
yywrap()
{
    return 1;
}


```

