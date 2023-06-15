So object-oriented view of a compiler, functional view of a compiler, what about a relational approach?

Let's suppose we have a simple language with just three kinds of expressions:
* Var s - Reference to a variable
* Apply e e -> Apply a function to an argument
* Lambda s e -> Create a function

We will consider a simple compiler pass on this, it checks whether any out-of-scope variables are used.

In an object-oriented language each expression type is represented by a class.

```
abstract class Expr:
  abstract string[] variables_used()
class Var(Expr):
  string variable_name;
  string[] variable_used():
     return self.variable_name
.. Similarly for Apply and Lambda
```

To write the phase to check whether any out-of-scope variables are used, we would have to create a method
`out_of_scope_variables` and implement it for each class. This means that adding a new compiler pass is relatively
because every class must be changed. Furthermore, if you expose your compiler inards as a library, say to
assist with editor tooling, then consumers either must be able to extend those classes, or work around that
in a less than appealing manner.

However, if you wish to add a new kind of expression, the experience is quite pleasant. You must implement
the class for your new expression and all the methods which must be implemented are clear in the abstract
base class.

In a functional language, this approach is somewhat turned inside-out. Instead you would create a single
type definition, in Elm called a custom type, but also called a custom union type, a union type, or a sum type.

```
type Expr =
     Var String
   | Apply Expr Expr
   | Lambda String Expr

variables_used : Expr -> Set String
variable_used e =
  case e of
    Var s -> Set.singleton s
    Apply left right -> Set.union (variables_used left) (variables_used right)
    Lambda s e -> Set.remove s (Set.variables_used e)
```

This is very pleasant for implementing a new compiler phase, but less great for implementing a new kind of
expression, because you have to touch all the existing functions. Personally I think the functional style here
is a clear winner, but it's possibly debatable and at the very least it does have some cons, even if those
are more than made up for with the pros.


What about a relational approach?

With a relational approach you need to decide what your tables are going to be. You could havea separate table for each kind of expression.
The problem with that, is that if you want a foreign-key for your 'parent', you would have to have one for each 
I am going to have a single expression table, with a single enum type for kinds of expresssions.
One possibility, is that you have a container 'expression' table, that just acts as a parent child facilitator. So, each expression kind has its
own table, with each row referencing a row in the generic 'expression' table, then they may **also** index into the same generic expression
table to reference their parents (or children).

The generic expression table could actually be a generic node, but then it can host other information, such as the string representing the
source code for that expression (or the source code locations).

There is no way to restrict the kind of expression, but then neither there is in the object-oriented or functional approaches.

But still finding the children of an expression is relatively tough, because you have to check in **all** the expression kind tables.
Unless you have an enum kind that tells you what kind of an expression it is. But in that case you're not getting a huge gain on the
functional style.


```
type ExprKind = Var | Apply | Lambda



```