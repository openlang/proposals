# Dynamic Lambda Syntax Proposal

##### Status: Draft
##### Submitted: 2016-07-22
##### Author: Robin Redeker
****************************

## Introduction

This is just an proposal to get the whole discussion started.

## Proposal

I have a more dynamic language in mind. One where the main functional
part comes from the lambda calculus. OO is handled by closures that
handle messages. The grammar is very simple to parse. Probably something
like LL(0) or LL(1). In that regard it's fairly similar to Lisp/Scheme.
In fact, in most regards it might be.

## Use cases

As source of ideas, as ignition to get others involved. To show that
it isnot hard to come up with some concrete ideas and a grammar.
It took me about 2 hours to set the basic idea and grammar up.
Another 30 minutes to learn EBNF.

## Implementation

### Examples

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


### EBNF Grammar

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
