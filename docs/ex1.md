YACC   

[![](img/logo.png)](index.html)

*   [Home](index.html)
*   [About](about.html)
*   [Roadmap](roadmap.html)
*   [Documentation](documentation.html)

Examples
--------

[Download](https://github.com/silcnitc/documentation/blob/master/yacc/yacc.odt?raw=true)

*   [YACC program](#navy)
*   [LEX program](#navl)

in2post.y

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
    

[top ↑](#navtop "Go back up")

in2post.l

	%{
		#include <stdio.h\> 
		#include "y.tab.h"
	%}
	%%
	\[0-9\]+	return DIGIT;
	'+'	return PLUS;
	'\*'	return STAR;
	'('	return \*yytext;
	')'	return \*yytext;
	%%
	yywrap()
	{
		return 1;
	}
      

[top ↑](#navtop "Go back up")

*   [Github](https://github.com/silcnitc)
*   [![Creative Commons License](img/creativecommons.png)](http://creativecommons.org/licenses/by-nc/4.0/)

[Go up](#navtop)

*   [Home](index.html)
*   [About](about.html)

window.jQuery || document.write('<script src="js/jquery-1.7.2.min.js"><\\/script>')