# Let-syntax for Proton

Introduction
============

This proposal will introduce `let` expressions in two forms:

1. As a way of introducing values into a local context
2. As a way of declaring an immutable reference to some value

1\) is referred to as the _expanded_ `let` expression or `let-in` expression,
and 2) is referred to as simply `let` expression or `let` declaration.

Proposal
========

This proposal will discuss the introduction of two forms of `let` expressions.
Both forms allow declaration of variables with varying scope.

For both the expanded `let` expression and the `let` declaration there is
the possibility of allowing multiple declarations in the `let` block.
I believe we should possibly start out with just allowing one declaration,
as the later addition of multiple declarations is rather trivial.
(On the paper and in the grammar, at least.)

Expanded `let`
--------------

Usually one would declare and use a value as such:

    double pi = 3.14;
    double pi2 = pi * 2;

After the declaration of `pi2` the value of `pi` lingers on and clutters
the namespace when declaring a new variable.
A `let` expression would be used to introduce scoping to a value declaration.
The above example would be rewritten such that the variable `pi` only
exist when declaring `pi2`.
Using the suggested syntax for `let` expressions, the above example
would be written as such:

    double pi2 = let pi = 3.14 in pi * 2;

Now, the variable `pi` exists only for the expression `pi * 2`.

### Relation to Expression
A `let` expression in this expanded fashion would be equivalent to any
other expression meaning that an expanded `let` expression could be used
for a `return` statement in a function as such:

    function Fun(b: Int) {
      return let a = 2 in a + b
    }

Simple `let`
------------

Using a `let` expression without the `in` block would declare the variable
in the inclosing scope; in the case of a function the scope would be the
function. In the case of an `if`-statement, the enclosing scope would be
the branch the `let` expression is declared in.

    function makeName() {
      let surname = "Doe"
      if (true) {
        let name = "John"
        return name + " " + surname
      } else {
        // name is out of scope here
        // so we can safely use the name again
        return let name = "Jane" in name + " " + surname
      }
      // name is out of scope here
    }

In the example above `surname` is in scope for the duration of `makePerson`
and `name` is in scope for the `true` branch of the `if`-statement.
Note that since `name` is not in scope in the `else` branch, we can safely
reuse the name here for our expanded `let` expression.

### Relation to Expression

A `let` declaration is **not** equivalent to any other expression meaning
that it **cannot** be used in a `return` statement.

Grammar
=======

~~~
let-expr        = "let", assignment, "in", expression
assignment      = identifier, "=", expression

let-declaration = "let", assignment

// Below is just the general declaration of identifiers
type-name       = capitalLetter, { anyLetter }
identifier      = lowercaseLetter, { lowercaseLetter }
expression      = let-expr
                | ...
~~~
