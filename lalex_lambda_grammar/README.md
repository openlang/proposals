Examples
========

Simple
------

    let add     = \a b -> $+ a b;
        my_sum  = $add ($add 10 20) 30; //=> 60
    $print $add my_sum 30;              //stdout=> 90

OOP
---

    let
        account_new = \ -> do (
            let balance = 10
            in \ cmd args* -> (
                if ($== cmd :deposit!)
                    ! balance = $+ balance @ 0 args
                elif ($== cmd :withdraw!)
                    ! balance = $- balance @ 0 args
            )
        );
        my_account = $account_new;
    in
        #$ :deposit! my_account 10;
    // or:
        ##$ :deposit! 10 my_account;
    // or:
        $my_account :deposit! 10;


EBNF Grammar
============

    expr-list       = expr , ";" , { expr , ";" } ;
    let             = "let" , { assignment ,  ";" } , "in" , expr-list ;
    func-expr       = expr ;
    arg-expr        = expr ;
    straight-call   = "$" , func-expr , { arg-expr } ;
    arg-swap-call   = ( "#" , "$" , arg-expr , func-expr , { arg-expr })
                    | ( "#", "#", "$" , arg-expr , arg-expr , func-expr, { arg-expr } )
                    (* every "." is for one prefixed argument.
                       theoretically unlimited dots possible *)
                    ;
    call            = straight-call
                    | arg-swap-call
                    ;
    identifier      = (* typical identifier *) ;
    function        = "\" , { identifier } , "->" , expr
    number          = (* some sensible number syntax *) ;
    variable        = identifier ;
    data-access     = "@" , expr , expr ;
                      (* first is index (numeric, or whatever),
                         second the vector or dictionary *)
    conditional     = "if" , "(" , expr , ")", expr ,
                      { "elif" , "(" , expr , ")" , expr } ,
                      [ "else", expr ] ;
    keyword         = ":" , identifier ; (* self evaluating keyword/symbol *)
    assignment      = "!" , identifier , "=" , expr ;
    comment         = ( "//" , (* any char until line end *) )
                    | ( "//[[" , (* anything *) , "]]" )
                    ;
    expr            = let
                    | call
                    | function
                    | number
                    | variable
                    | conditional
                    | data-access
                    | keyword
                    | assignment
                    | comment
                    | "(" , expr , ")"
                    | "do" , "(" , expr-list , ")"
                    ;
