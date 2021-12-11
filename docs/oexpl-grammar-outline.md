---
title: 'OExpL Grammar outline'
hide:
    - toc
    - navigation
---

# Annotated OExpL Grammar outline

An outline for the OExpL grammar is given here. Calls to functions that update the symbol table, type table, class table and the abstract syntax tree data structures are indicated as semantic actions at certain places.

=== "1. Program"

=== "TypeDefBlock"

    ``` c
    TypeDefBlock  : TYPE TypeDefList ENDTYPE
                  |
                  ;

    TypeDefList   : TypeDefList TypeDef
                  | TypeDef
                  ;

    TypeDef       : ID '{' FieldDeclList '}'   { Tptr = TInstall(tname,size,$3); }
                  ;

    FieldDeclList : FieldDeclList FieldDecl
                  | FieldDecl
                  ;

    FieldDecl    : TypeName ID ';'

    TypeName     : INT
                 | STR
                 | ID       //TypeName for user-defined types
                 ;
    ```

=== "ClassDefBlock"

    ```c
    Program : TypeDefBlock ClassDefBlock GlobalDeclBlock FuncDefBlock MainBlock
    ;
    ClassDefBlock : CLASS ClassDefList ENDCLASS
                    |
    ;
    ClassDefList : ClassDefList ClassDef
                   | ClassDef
    ;
    Classdef      : Cname '{'DECL Fieldlists MethodDecl ENDDECL MethodDefns '}'
    ;
    Cname         : ID        {Cptr = Cinstall($1->Name,NULL);}
                  | ID Extends ID {Cptr = Cinstall($1->Name,$3->Name);}
    ;
    Fieldlists  : Fieldlists Fld
            |
    ;
    Fld         : ID ID ';'  {Class_Finstall(Cptr,$1->Name,$2->Name);} //Installing the field to the class
    ;
    MethodDecl : MethodDecl MDecl
           | MDecl
    ;
    MDecl      : ID ID '(' Paramlist ')' ';' {Class_Minstall(Cptr,$2->Name,Tlookup($1->Name),$4);}
                                                //Installing the method to class
    ;
    MethodDefns : MethodDefns FDef
                | FDef
    ;
    stmt :    ...
              | Field ASSIGN Expr ';'
              ...
              | ID ASGN NEW '(' ID ')' ';'
              | Field ASSIGN NEW ’(’ ID ’)’ ';'
              | DELETE ’(’ Field ’)’ ';'
    ;

    Expr:     ...
              | Field
              | FieldFunction
    ;
    Field:    SELF '.' ID
              |ID '.' ID   //This will not occur inside a class.
              |Field '.' ID
    ;
    FieldFunction : SELF '.' ID '(' Arglist ')'
                    |ID '.' ID '(' Arglist ')'   //This will not occur inside a class.
                    |Field '.' ID '(' Arglist ')'
    ;
    Arglist: Arglist ',' Expr
              |Expr
              |
    ;
    ```

=== "GDeclBlock"

    ```c
    GDeclBlock : DECL GDeclList ENDDECL
               |
               ;

    GDeclList  : GDecList GDecl
               | GDecl
               ;

    GDecl      : TypeName Gidlist ';'
               ;

    Gidlist    : Gidlist ',' Gid
               |   Gid
               ;

    Gid        :   ID                      { GInstall(varname,ttableptr,ctableptr, 1, NULL); }
               |   ID '(' ParamList ')'      { GInstall(varname,ttableptr,NULL, 0, $3);   }
               |   ID '[' NUM ']'          { GInstall(varname,ttableptr,NULL, $3, NULL);   } //Arrays are not allowed with class objects.
               ;

    ParamList    :  ParamList ',' Param  { AppendParamlist($1,$2);}
               |  Param
               |  //There can be functions with no parameters
               ;

    Param        : TypeName ID { CreateParamlist($1,$2); }
               ;

    ```
    !!! note
        1. The second argument to the function `Ginstall()` must be a pointer to a type table entry which will be `NULL` if it is of class type.
        2. The third argument to the function `Ginstall()` must be a pointer to class table entry which will be `NULL` if it is not of class type.
        3. The functions `CreateParamlist()` and `AppendParamlist()` help to create a linked list containing the types and names
          of parameters specified in an ExpL function declaration. Design of these functions is left to you.

=== "FDefBlock"

    ```
    FDefList  : FDefBlock
              | FDefList FDefBlock
              ;
    FDefBlock : TypeName ID '(' ParamList ')' '{' LdeclBlock Body '}'  { GUpdate($2->name,$1,$4,$7,$8); }
              ;

    Body      : BEGIN Slist Retstmt END
              ;

    Slist     : Slist Stmt
              |
              ;

    Stmt      : ID ASGN Expr ';'
              | ....
              | IF '(' Expr ')' THEN Slist ELSE Slist ENDIF ';'
              | ...
              | ID ASGN ALLOC'(' ')' ';'
              | FIELD ASGN ALLOC'(' ')' ';'
              | FREE '(' ID ')' ';'
              | FREE '(' FIELD ')' ';'
              | READ '(' ID ')' ';'
              | READ '(' FIELD ')' ';'
              | WRITE '(' Expr ')' ';'
              ;

    FIELD     : ID '.' ID
              | FIELD '.' ID
              ;

    Expr      : Expr PLUS Expr   { $$ = TreeCreate(TLookup("int"),NODETYPE_PLUS,NULL,(union Constant){},NULL,$1,$3,NULL); }
              | ....
              | '(' Expr ')'
              | NUM
              | ID
              | ID '[' Expr ']'
              | FIELD
              | ID '(' ArgList ')'  {
                                    gtemp = GLookup($1->name);
                                    if(gtemp == NULL){
                                        yyerror("Yacc : Undefined function");exit(1);
                                    }
                                    $$ = TreeCreate(gtemp->type,NODETYPE_FUNCTION,$1->name,(union Constant){},$3,NULL,NULL,NULL);
                                    $$->Gentry = gtemp;
                                  }
              ;
    ```

=== "MainBlock"

    ```c
    MainBlock : INT MAIN '(' ')' '{' LdeclBlock Body '}'
                                    {
                                        type = TLookup("int");
                                        gtemp = GInstall("MAIN",type,0,NULL);
                                        //...Some more work to be done
                                    }
              ;
    ```
