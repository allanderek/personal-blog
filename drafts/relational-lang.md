What would a relational language look like, if we used it to actually produce HTML or JSON?
This goal here would be to make a backend language for web applications, that doesn't have any problem storing things in relational format.


The first thing is can we deal with tree data structures and their formatting?

Creating a tree structure is fine:

insert divId <- div
 insert class "container" divId
 insert content "I'm a div" divId


What about formatting this:

formatRow row :
 "<" + row.kind 
 for attribute in attributes where parentId <- row.id
 + ">"
 for content in elements where parentId -< row.id
    formatRow content
 + "</" + row.kind ">"

  
## Homioiconic

## Meta programming

From concatenative.org

"Higher-order concatenative languages, such as Joy and Factor, represent quotations (anonymous functions) as lists. 
This means that any list manipulation operations can be used to construct new programs and run them."

So I wonder if a relational language can do the same?

## Dependent types

As I understand it, dependent types allow meta-programming.
So can we do something like, take in a union type and produce all the constructors?
Or take in a union type and produce a decoder?

## Iterators / Laziness

I've also been a big fan of laziness, but I do understand the difficulties it brings. I also like the Wadler style 'listlessness is better than laziness'.
Because of this, I've been a big fan of python-style iterators, which I think are pretty close to providing some form of laziness.
Can we also do this kind of thing.

### Iterating / querying

With this in mind, one of the main things we're trying to avoid here, is returning a larger than necessary table. So where the whole object/relational
mapping goes awry, is if you some kind of transformation/filter that is authored in the **object** style, you cannot apply that during
the query. So for example, suppose you want to return a list of all users who have logged-in, in the last month. That's an easy enough query,
but if you're using an ORM, you have to be careful that you do not query for all the users, marshall those into a 'user' objects, and then
call a `user.loggin_in_recently()` method on each one, in a `List.map .logged_in_recently all_users` style. This would be very inefficient.

Now, if the filter/result is more complicated, it's more likely that you have/want to have the logic in the **object** language. For example your
query might be 'return all users with a weak password', and the problem is your code for determining whether a password is weak is written in
your object language, not SQL. So now you have to either translate that logic (and maintain the translation) into SQL, or you're going to have a
slow query.

The goal here, is to have your general language be relational, such that your password evaluation code is already in the relational language so
you can write your query in a natural way.

(note: Obviously the passwords aren't going to be in plain-text so this is perhaps a bad example, but you get the point).



## Sub-typing

Can we increase composibility, by having types on code that means it works over more than one kind of table. So for example, maybe you have a
'function' that sums the age column, so you can pass that any table that contains an 'age' column (assuming the 'age' is an 'int')


## Typing

Let's start with just some typing in general, let's just write down some code and try to type in a hypothetical relational language.
I think values could have one of several main kinds:
1. A primitive value such as an integer.
2. A row of primitive types
3. A table of rows
4. Functions or queries that operate over those.


Let's suppose we only have `Int`, and `String` as primitive types.
A row type, is simply like a record type: `{ name : String, age : Int}`
A table type, is simply that there are multiple records all of the same type.
We will probably have a `foreign-key` type that references into a table, but the type of the `foreign-key` should simply be the table type
which can be parameterised over things which are not referenced. So we can have a function/query that operates over any table that has an age column which is an integer.
One large problem is how to deal with custom union types. I think this should be part of the row type. But I'm not stuck on that.

We might have a `Maybe Row` type meaning, 1 or 0. That would be the result of, for example, a maximum, or find query.

So let's type some queries/expressions?

1 : Int
{ name : "Allan", age : 42 } : { name : String, age : Int }
[ { name : "Allan", age : 42 } ] : Table { name : "Allan", age : 42 }
filter age > 18 : Table { a | age : Int} -> Table { a | age : Int }

We need generative statments, such as how to produce the list of integers?
This is quite an important part, this is the bit that is like Python iterators. Perhaps we need functions:

def integers x y =
 case x < y of
  True ->  yield x and integers (x+1) y
  False -> end
 
Something like this.

Possibly saving idea for this, you can have custom types, they just cannot contain other custom types. Just a row types cannot contain
other row types. So this means that are values are always 'flat'. In this way I think we can implement an abstract syntax tree in the
way that we would want.

So we ultimately end up with a small functional language which has row/record and custom union types, but neither can contain themselves
or the other, so you can only have flat types. A custom union type is just a row type that can have different fields depending on the first one.
We then have to figure out how we're going to do iterators.

So ultimately you have iterators, and those iterators iterate over a sequence of:
1. Primitive types, so Iter Int
2. Row types, so Iter { age : Int, naem : String }
3. CustomTypes, so Iter Color

Now let's try to implement an abstract syntax tree:

So each row is an custom type, which represents an expression.


# Examples

Can we 'build our own text editor' https://viewsourcecode.org/snaptoken/kilo/, pretty sure there are versions of this Rust etc.

## Meta-Typing

What if the user was able to define their own type system. I'm not saying they could then make it only possible to,
for example create an AST of a HM typeable expresssion, but they could make it impossible to create the AST of an
invalid expression. So for example, maybe apply has exactly two sub-expressions.
