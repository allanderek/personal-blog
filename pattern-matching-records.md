---
title: "Pattern matching records"
tags: programming elm
date: 2021-02-19
---

A [blog post](https://buttondown.email/nelhage/archive/tagged-unions-are-overrated/) popped up on my feed, which has the main point that tagged union types (called *custom types* in Elm) are overrated for implementing intermediate representations. To be clear the author is not saying that tagged union types are in general overrated, but their suitability for intermediate representations within compilers is just not that big of a win.

It's easy to see why tagged union types are thought of as perfect for internal representations of code in compilers. The grammar of a language kind of looks like a tagged union type already. You might represent the grammar of an elm expression in Elm's custom types something like the following:

```elm
type Expr
    = IntLiteral Int
    | FloatLiteral Float
    | StringLiteral String
    | Name String
    | FunctionApplication Expr (List Expr)
    | LetExpr (List LetDefs) Expr
    | CaseExpr Expr (List (Pattern, Expr))
    | RecordExpr (List (Name, Expr))
```

Missing a couple of expressions such as record update expression but you get the idea. This can indeed form the basis of your type for the abstract syntax tree, but **typically** you will need to store much more information. One problem is that much of the information you wish to store is the same for all types of expressions. This means that you end up with either a record type in which one field is the tagged union type, **or** you stuff all the information into each constructor of the tagged union type.

For example, when you detect an error in the program you will want to print out the offending expression, or at least some of it. So you need to store here some representation of the actual source code of the expression in question. There are a couple of ways of doing this, you can either just directly store the string, which ends up using quite a bit of memory because you store the same parts of the string over and over again as you parse down the tree. Alternatively, you can just store the start and end character positions of the expression, because for most of them you will not need it, when you need to print out an error, you can recover the original string. Either way lets assume you have some type `SourceCode` that relates to the source code of a particular expression. We can update our abstract syntax tree in one of two ways, first of all with a record type:


```elm
type alias Expr =
    { source : Sourcecode
    , tree : ExprTree
    }
type ExprTree
    = IntLiteral Int
    | FloatLiteral Float
    | StringLiteral String
    | Name String
    | FunctionApplication Expr (List Expr)
    | LetExpr (List LetDefs) Expr
    | CaseExpr Expr (List (Pattern, Expr))
    | RecordExpr (List (Name, Expr))
```

Note that these are mutually recursive types, inside the `ExprTree` type we reference `Expr` and inside the `Expr` type we reference `ExprTree`. The alternative is to push the source code each of the constructors:


```elm
type Expr
    = IntLiteral SourceCode Int
    | FloatLiteral SourceCode Float
    | StringLiteral SourceCode String
    | Name SourceCode String
    | FunctionApplication SourceCode Expr (List Expr)
    | LetExpr SourceCode (List LetDefs) Expr
    | CaseExpr SourceCode Expr (List (Pattern, Expr))
    | RecordExpr SourceCode (List (Name, Expr))
```

The downside of the latter is that you cannot refer to `expr.source`. In the case that the meta-data you're intersetd in is the original source code of the expression, you probably only use that when printing out an error message, so it's probably not *that* much pain. However, a compiler might end up with quite a bit of extra information, such as the names used in an expression, its inferred type etc. Each of these bits of meta data might be used more frequently, still it's not that much pain to call a function to get some meta data rather than access it via a record field access.

The downside of the former approach is that pattern matching deeper than the first layer is awkward, or, as in Elm, not at all possible. Unfortunately transformations inside compilers can often use nested patterns quite usefully. For example, suppose I want to take all (fully applied) uses of `Maybe.withDefault` and transform them into `case` expressions (so that the default value is never needlessly calculated), so I want to turn:

```elm
Maybe.withDefault (createBlankUser model) mUser
```

into:

```elm
case mUser of
    Nothing ->
        createBlankUser model
    Just user ->
        user
```

Using the latter form I can do the pattern match like so:

```elm
case expr of
    (FunctionApplication 
        applSource
        [ Name _ "Maybe.withDefault"
        , defaultExpr
        , maybeExpr
        ]) ->
        CaseExpr 
            applSource
            maybeExpr
            [ ( NamePattern "Nothing", defaultExpr)
            , (TaggedPattern "Just" (NamePattern "expr"), Name "expr")
            ]
    ... recursively apply this transformation ...
```

I've fudged this a little because we didn't specify an abstract syntax tree type for the patterns but the ones we are producing should probably have source information, which would probably have to be some value that indicates there is no source because the pattern is a result of an internal transformation. Anyway the point here is that I can do this kind of nested pattern matching to find a tree of the right *'shape'*. The key part of the above code is the nested pattern `(FunctionApplication applSource [ Name _ "Maybe.withDefault, defaultExpr, maybeExpr])`, this is a pretty neat way of saying *"Match all function applications that have three sub expressions, the first of which is the name expression 'Match.withDefault'"*. 

Now with the latter outer record type representation we cannot do this, because we cannot pattern match on record expressions (in Elm), you have to do `case expr.tree of`, so you get some less nice code:


```elm
case expr.tree of
    (FunctionApplication [ fun, defaultExpr, maybeExpr ]) ->
        case fun.tree of
            Name "Maybe.withDefault" ->
                { source = expr.source
                , tree = 
                    CaseExpr
                        maybeExpr
                        [ ( NamePattern "Nothing", defaultExpr)
                        , (TaggedPattern "Just" (NamePattern "expr"), Name "expr" )
                        ]
    ... recursively apply this transformation ...
```

I quite like this representation. It would **perhaps** be a little nicer if I could do:

```elm
case expr.tree of
    (FunctionApplication [ { tree = Name "Maybe.withDefault" }, defaultExpr, maybeExpr ]) ->
        { source = expr.source
        , tree = 
            CaseExpr
                maybeExpr
                [ ( NamePattern "Nothing", defaultExpr)
                , (TaggedPattern "Just" (NamePattern "expr"), Name "expr" )
                ]
    ... recursively apply this transformation ...
```

In reality, and I think this is the main point behind the linked to blog post, neither of these patterns quite work. You cannot really pattern match in this way, because instead of asking if the function being applied is literally written as `Match.withDefault`, we want to know if that is the function being applied. We have to take care of at least two scenarios. The first is that you have imported `withDefault` unqualified with `import Maybe exposing (withDefault)`, although if you do not catch this case nothing bad happens it is a common way to import that function. The second scenario would be way less common, but actually mean your optimiser could introduce a bug. That is where you have made an alternative `Maybe` module, that exports `withDefault`, and you have imported this alternative whilst aliasing it to `Maybe` a la `import Helpers.Maybe as Maybe`. Presumably your `withDefault` does a different thing, so the optimisation in question wouldn't be appropriate. Kind of unlikely, but you have to handle such a case. A more common, related case, is if you allow the unqualified `withDefault` then what about if someone has imported `import Result exposing (withMaybe)`. A final possibility is `import Maybe as M` and then used `M.withDefault`. Note that in many of these cases you basically need to know the current scope. So you simply won't be able to make this test using a pattern. 

The point is, that in reality the ability to nest patterns like this is not that useful, because ultimately the patterns you want to match against are just not that simple. 

## Polymorphism

One further point in this somewhat rambling post. Sometimes in a compiler you will want to 'add' meta-data to the abstract syntax tree as you complete passes over the compiler. In the non-record type this is quite elegant, you can do the following:

```elm
type Expr a
    = IntLiteral a Int
    | FloatLiteral a Float
    | StringLiteral a String
    | Name a String
    | FunctionApplication a Expr (List Expr)
    | LetExpr a (List LetDefs) Expr
    | CaseExpr a Expr (List (Pattern, Expr))
    | RecordExpr a (List (Name, Expr))

type alias ParsedExpr =
    Expr { source : SourceCode }
```

Then you can write a pass for the compiler that changes the type of the meta-data, maybe it adds meta-data:

```elm
type alias NamedExpr =
    { source : SourceCode
    , names : List Name
    }
usedNames : ParsedExpr -> NamedExpr
usedNames expr =
    case expr of
        IntLiteral meta i ->
            let
                newMeta =
                    { source : meta.source
                    , names = []
                    }
            in 
            IntLiteral newMeta i
        ...
        Name meta s ->
            let
                newMeta =
                    { source : meta.source
                    , names = [s]
                    }
            in 
            Name newMeta s
        ...
```

Or you could write a pass that throws some meta data away. For example, once you have type checked the program, you know you won't need the 'source' anymore, so now you can write passes such as the one detailed above that transforms the source, only now, any **generated** expressions do not need to add in 'source' code, which is great because obviously such generated expressions do not have a representation in the original source code.

Using the record-based expression representation it's a little more awkward to allow for different types of metadata.

```elm
type alias Expr a =
    { meta : a
    , tree : ExprTree
    }
type ExprTree a
    = IntLiteral Int
    | FloatLiteral Float
    | StringLiteral String
    | Name String
    | FunctionApplication (Expr a) (List (Expr a))
    | LetExpr (List (LetDefs a)) (Expr a)
    | CaseExpr (Expr a) (List ((Pattern a), Expr a))
    | RecordExpr (List (Name, Expr a))
```
