# Record syntax for Proton

Introduction
============

This proposal will introduce the concept of records along with
two new keywords `record` and `with`.

What is a record?

A record is a collection of data. The prime example is a person.
A person has a name and an age so in other words the tuple `(<name>, <age>)`
could represent a person.
To give name to this representation, we collect the data in a record.

As records are a collection of values used to represent something, they
inherently need to be equatable. That is, we need to be able to compare
two instances of the same record type (e.g. two persons).

A record in Proton is immutable. That is, once it has been declared
there is no way to mutate the data held in an instance of said record.
Ways of dealing mutating values stored in a record will be presented
though these are subject to change based on future proposals (specifically
closures from @lalex).

### Assumptions

I am making the following assumptions:

* Type names start with a capital letter (e.g. `Person` instead of `person`)
* Variables and identifiers start with a lowercased letter
* The type goes on the right side of an expression. (`age: Int` instead of `Int age`)
* Instantiation does not require `new`
 
Some of these assumptions might prove wrong in which case I will update
this list accordingly. Also some might prove to be an inspiration for
future proposals.

### Features

Just for clarity, the features outlined in this proposal is the following:

1. Declaration of a record
2. Generation "magic" for records

Proposal
========

Declaration
-----------

Declaring a record should ideally be as simple as possible.
I propose to introduce `record` as a keyword.
Like in the following example:

    record Person(name: String, age: Int)

The part `(name: String, age: Int)` is referred to as the _record constructor_.
Declaring a record would generate a function with the name of the record
taking the number of parameters declared in the record constructor.

Declaring a record would also generate accessors for each given member
declared in the record constructor.
As in the example with `Person` both `name` and `age` would be members of
the `Person` record and accessors would be created for both members.

Usage
-----

Instantiating a record is done via the function generated when the record
was declared.
Thus to create an instance of the `Person` record, we would use the
following form:

    Person("John Doe", 99)

### Member Access

Accessing a member of a record would be possible using dot-syntax.
That is `record.memberName`.

As the member of a record might be a collection or some other type
that has members itself, this member access is going to be an expression
such that further "dotting" is possible.

### Mutation

**NOTE:** This might need to go in its own separate proposal.

Since records in this proposal is immutable we can't directly
mutate values.
We need some way to mutate and update values in a record.

I propose to introduce the keyword `with` for mutating one or several
members of a record.
See below example.

    let person1 = person with { age = 5 } // Will create a new instance of Person with age set to 5
    let person2 = person with { age = 5, name = "Jane" }

#### Internal Implementation

Taking a note from the C# design team, we could let the `with` expression
compile to a function call on the record.
This function (let's call it `With` with a capitalized "w") would be a
function whose parameters all had their default values set to the actual
values for the record instance.

Thus, for the `Person` record -- declared below -- the `With` function 
would look like this:

    let p1 = Person("Jane Doe", 12)
    With(name: String = "Jane Doe", age: Int = 12) { ... }

This would translate the following:

    let p2 = p1 with { name = "John", age = 5 }
    let p3 = p2 with { name = "Shaun" }

into this:

    let p2 = p1.With("John", 5)
    let p3 = p2.With("Shaun", 5)

This way of doing it probably uses positional arguments and I don't know
whether we want to go with named or positional arguments for optional
parameters.
This part of the proposal will most likely have to wait until we decide
which of the two we want to pursue.

"Magic"
-------

As records will most likely be used for representing a collection
of values we need to ensure that there exists a straightforward way
of testing for equality.

Thus along with a record declaration comes:

* a compiler-generated method testing for equality
* a method returning the unique hashcode for the instance

The equality for a record is based on its members meaning that iff two
instances have the same value for each and every member, then the two
instances are considered equal.

~~~
let p1 = Person("Name", 0)
let p2 = Person("Name", 1)
let p3 = p2 with { age = 0 }

p1 == p2 // False!
p2 == p3 // False!
p1 == p3 // True!
~~~

Grammar
=======

~~~
record-declaration  = "record", type-name, "(", member-declarations, ")"
member-declarations = member-declaration, { ",", member-declaration }
member-declaration  = identifier, ":", type-name

member-access       = type-name, ".", identifier

with-expr           = type-name, "with", "{", assignment-list, "}"
assignment-list     = assignment, { ",", assignment }
assignment          = identifier, "=", expression

// Below is just the general declaration of identifiers
type-name           = capitalLetter, { anyLetter }
identifier          = lowercaseLetter, { lowercaseLetter }
expression          = ...
~~~
