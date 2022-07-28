---
title: "Let-in Let-in"
tags: elm programming
date: 2021-03-03
---

I noticed a sort of surprising syntax thing in Elm yesterday. When you think about it, it's not all that surprising, but the question is, can it be made of use? So what I noticed is that the `in` expression of a `let-in` expression can itself be a `let-in` expression. So the following is valid syntax:

```elm
...
    let
        x = 1
    in
    let
        y = 2
    in
    x + y
```

That's because the syntax of `let-in` is `let <def-list> in <expr>` and `let-in` is itself a valid `<expr>`. So you can have as many `let-in` blocks chained as you see fit. Clearly, you can just delete the middle two `in-let` lines and you still have a valid expression. The question then is, what is  this useful for?

Not much I guess, the only difference is that now definitions in the top-block cannot see any of the names defined in the bottom block, so whilst this is still valid:


```elm
...
    let
        x = 1
    in
    let
        y = x + 2
    in
    x + y
```

The following is not valid, because `y` is not in scope in the upper definition block:


```elm
...
    let
        x = 1 + y
    in
    let
        y = 2
    in
    x + y
```

Could you use this? I've [written before](/posts/2021-02-17-missing-language-feature) about the missing language feature to remove a name from the scope. It would be nice to be able to use this in such a fashion. That missing feature is to allow you to update the model without fear of accidentally using the old (non-updated) model. As in:

```elm
update message model =
    case message of
        SomeMessage ->
            let
                newModel = 
                    ... <expr> involving model

            in
            ( { newModel | status = InFlight }
            , requestEntries newModel
            )
        ...
```

In this example the defined `newModel` name is used in the `in` expression twice, a common bug is to accidentally use `model` where `newModel` is correct. Unfortunately using multiple `let-in` blocks chained does not help us solve this problem. I could possibly see a case where certain expressions need to be sure to use the **old** `model`, rather than the new `newModel` in which case chaining `let-in` expressions might help with that. I think it's a pretty marginal use though.
