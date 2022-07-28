---
title: "Frugal syntax formatter"
tags: elm syntax
date: 2021-01-27
---

As I've mentioned many times on this blog Elm has a famously pretty frugal syntax. There are obviously positives and negatives from this. A potential negative is that you simply cannot write the code the way you would wish to, which presumably might mean that the code you **do** write is inferior, since you obviously had some reason for wishing to write it another way. Still, in general I think the positives outweight the negatives, and have even argued in the pass that Elm could consider removing some syntax, specifically lambda expressions and if-then-else expressions. Though it must be said I've also argued for the inclusion of the odd bit of *new* syntax.

One benefit a frugal syntax has, that the parser for the compiler can be made a bit simpler. But it's not only the compiler that benefits from this, there are a bunch of other tools that accompany a language. Such as an editor plugin, which usually has to parse the language, a code-formatter, any static analysis tool, and a documentation generator tool. It's unfortunate as well that these tools often cannot share a parser. For example the editor-plugin may have to be written in a language that is not the same as the compiler. The code-formatter and documentation-generator often need to parse in a different way because they need to retain information (such as the comments, and the original white space) that the compiler throws away.

An additional benefit is related to decisions that have to be made about how to format your code. The Elm community, along with some other language communities, have embraced the idea that if everyone is forced into using an automated code-formatter the fact that the decision is mostly taken away from you as a developer is rather liberating and frees your analytic mind up to more important tasks. However, even if you all use the same code-formatter, the code-formatter itself must decide how to format code. At first this may seem relatively trivial, just take each kind of node in the grammar of the language and decide how it ought to be formatted. But, how different forms combine together can be challenging. With a more frugal syntax, there are less ways to combine different forms and the code formatter, can both be simpler and more likely usable.

One element of the Elm syntax that has been a minor pain point is the expression before the `|` in a record update expression. In the Elm grammar this can only be a single name. It cannot be a generic expression. In particular it cannot be a function application. This means the formatter knows how to format record update expressions. If there is only one field being updated, it is on the same line:

```elm
{ myRecord | x = 10 }
```

If either the expression to which the field is assigned is complicated, or there is more than a single field being updated, then we take a new line for each field, all indented one indentation more than the record update expression itself:


```elm
{ myRecord 
    | x = 10
    , y = 20
    }
```

and similarly


```elm
{ myRecord 
    | point = 
        { x = 10
        , y = 20
        , z = 30
        }
    }
```

Great. No arguments here, even if you would have a slightly different preference you will be glad of the enforced uniformity. The fact that Elm does not allow an arbitrary expression before the `|` means that we do not have to consider how to format an interesting expression there:

```elm
{ update
    a
    function
    with
    a
    number
    of
    arguments
    | x = 10
    , y = 20
    }
```

Simply cannot happen, and we therefore do not need to consider this possibility.  In general a more complicated grammar presents more possibilities to disagree about formatting.

p.s. I still think the Elm grammar should be relaxed here and at least allow module access before the bar and possibly record access. So basically names with dots in them:

```elm
{ Init.record | x = 20 }
{ myRecord.point | x = 20 }
```


