---
title: "Horizontal and Vertical"
tags: programming
date: 2021-03-12
---

I wrote a little about [Tailwind CSS](/posts/2021-03-06-tailwind/) and then up on my feed popped an [article which argues against tailwind](https://www.aleksandrhovhannisyan.com/blog/why-i-dont-like-tailwind-css/). There are of course many reasons why two people can see what seems like the same information and reach different, and possibly entirely opposite, conclusions. But one quite obvious reason, when we're talking about choices of technology, is that both have different advantages and disadvantages, and for some reason the two people are weighing those differently. Often it means that there is no *'right answer'* in general, and unfortunately no way to combine the advantages of both. I sometimes think of this in terms of horiztonal and vertical. In this post I'm going to try to give an example of this distinction.

Suppose you're writing a compiler for a language. Let's say it has the following kinds of expressions:
1. A literal expression (normally floats, ints, and strings, but let's keep things simple and just have integers).
2. Variable use
3. Lambdas
4. Function application
5. let bindings, consisting of a bunch of definitions and a single expression, where a single definition consists of a pair of a name and an expression.

How to model this in your programming language? Functional languages model this quite directly using *custom tagged types*, like so:

```elm
type alias Name = String
type Expr
    = Literal Int
    | Variable Name
    | Lambda Name Expr
    | Application Expr Expr
    | Let (List Definition) Expr
type alias Definition =
    { name : Name
    , expr  : Expr
    }
```

In an object-oriented language you model this by having a base (possibly abstract) class, followed by classes for each of the subsequent expression types:

```python
class Expr (object):
    pass

class Literal (Expr):
    def __init__(self, value):
        self.value = value
class Variable (Expr):
    def __init__(self, variable):
        self.variable = variable
class Lambda (Expr):
    def __init__(self, name, value):
        self.name = name
        self.value = value
...
```

At this point, I must admit that the functional style does seem just obviously better. However, you will have to actually define some 'passes' over your data structure. Various things you will wish to do with your compiler, you might have a simplification stage, a typing stage etc. The first thing you might do is a pass which reports the names used in an expression. In a real compiler your expression representation would have some knowledge of the source code location of that expression, but let's ignore that for now, suppose you just want to know which names are used because you ultimately wish to work out whether there are any names which are defined but not used. This is actually computing the set of *'free variables'* that is, the set of names that are used by an expression without being defined by that expression. So in other words `\\x -> x + a` would have the `a` as a free variable, but not `x`.

```elm
freeVariables : Expr -> Set Name
freeVariables expr =
    case expr of
        Literal _ ->
            Set.empty
        Variable name ->
            Set.singleton name

        Lambda arg  body ->
            freeVariables body
                |> Set.remove arg
        Application f a ->
            Set.union (freeVariables f) (freeVariables a)
        Let definitions body ->
            let
                namesDefinedHere =
                    List.map .name definitions
                        |> Set.fromList
                namesUsedHere
                    List.map .expr definitions
                        |> List.map freeVariables
                        |> List.foldl Set.union Set.empty
            in
            Set.diff namesUsedHere namesDefinedHere

```

So in order to define this pass, we do not have to touch the type definition. The object-oriented approach however does touch the modelling of the data type:

```python
class Expr (object):
    pass
class Literal (Expr):
    def __init__(self, value):
        self.value = value
    def free_variables(self):
        return set()
class Variable (Expr):
    def __init__(self, variable):
        self.variable = variable
    def free_variables(self):
        return { self.variable }
class Lambda (Expr):
    def __init__(self, name, value):
        self.name = name
        self.value = value
    def free_variables(self):
        s = self.value.free_variables
        s.remove(self.name)
        return s
...
```

This means that in the object-oriented style if you want to add a **pass** over the syntax of expressions, you need to touch the type definitions of all kinds of expressions you have. In the functional approach you do not need to touch anything that has already been defined. I think this makes defining a new pass over  the syntax of expressions more convienent and intuitive. 

However, what if you want to add a new kind of expression? In the functional style, you have to touch the type definition **and** all previously defined functions that operate over that type. In the object-oriented style you do not have to touch anything that has previously been defined, you just build a new class and implement each of the required methods for that new class.

So which is better? Well I think for compilers I have a clear preference for the functional style, but the point is that both styles have somewhat opposite advantages/disadvantages. If you release this as a library, in the functional style, users would find it easy to extend with new passes over the expression type, but pretty difficult to extend the kinds of expressions. If you release this as a library in the object-oriented style users will have a nicer time extending the kinds of expression available but awkward to write new passes. 

So if you frequently encounter problems, like writing compilers, that lends itself well to the functional approach, you'll probably prefer functional languages. However if you commonly encounter problems that lend themselves well to extension of the type, rather than the operations around that type, you may well have a preference for object-oriented languages.

Can you have the best of both worlds? Not really, even if your language, like O'caml, supports both classes and custom tagged union types, you still have to decide which datastructure to use to represent your data. In the example shown, you still have to represent expressions as either a custom tagged union type, or an extensible class. So either adding a pass over that data type, or extending that datatype is going to be more pleasant than the other.
