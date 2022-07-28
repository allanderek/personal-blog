---
title: "Elm and lambda expressions"
tags: elm syntax
date: 2021-01-24
---

I have remarked before that Elm is pretty frugal when it comes to syntax. Lambda expressions though are a sort of staple of functional languages. For no particularly good reason a language is not seen as functional if it doesn't have lambda expressions. It's even possible to see phrases attributing elements of functional programming that have found their way into mainstream imperative languges such as Python, and it often includes lambdas. So lambda expressions are somehow seen as a quintessentially *functional* feature.

I think I could live without them. It's always the case that you can simply introduce a `let` declaration and actually give a name to the function however small. So the question becomes is a lambda ever more readable than a defined function name? I've had a look around and there are certainly times when lambdas are over-used, resulting in a kind of code-golf style that would definitely benefit from a name. The remaining times I think that the lambda is fine, but not particularly *better* than a `let` defined named function.

On the other  hand, naming things is long claimed to be one the two hardest things in computer science, so avoiding it altogether could conceivably increase productivity. I remain sceptical.

So overall, I think occasionally a lambda is convenient, but there are already a couple of places in the grammar where Elm decides frugality is more important than convenience, for example, record update expressions in which the expression being updated must be an unqualified variable name. So lambdas seem at least inconsistent.

To make the removal of lambdas a little more palatable, I think we could include some more syntactic sugar. We already have: `.field` as a sugar for `\r -> r.field`. I think this should be extended to allow any number of chained field accesses `.one.two.three` as sugar for `\r -> r.one.two.three` (currently you can avoid the lambda for this by using `(.one >> .two >> .three)` but I think my suggestion is neater. Finally I would include pipeline friend record update, using a new character such as `x!y` to be sugar for `\r -> { r | x = y }`. 

I admit I don't have much good argument for it, but my feeling is both of these additions would be better than general lambdas.
