---
title: "Let-in, where, and the right pizza"
tags: programming
date: 2021-03-18
---

When I started programming in Elm I was already pretty fluent in Haskell and quite used to using the `$` operator, which in Elm is written as `<|` and sometimes called the *'left pizza'* operator. Somehow, over time I started to use the `|>` (*right pizza*) operator more and more.

I think this first came about through the use of the [builder pattern](/posts/2021-01-02-builder-pattern) which very naturally maps the building up of a data structure which thus just got me comfortable with post-fix application of a function. 

Once you're comfortable with it, I often find the use of the `|>` operator allows you to put the *important* thing first. For example, I might write something like the following:

```elm
Html.label
    []
    [ Html.text (String.toLower user.email) ]
```
The problem with this is that the important thing here is *what* you are displaying, that is, the user's email address, not really *how* you display it. The details of *how* you display that are only really important when you're concentrating on that part alone, in which case it probably doesn't matter much in which direction it is written. So I find the following a *marginally* better:


```elm
Html.label
    []
    [ user.email |> String.toLower |> Html.text ]
```

Here the important thing, the *what* you're displaying is *first*. Some people also quite like this style because you're chaining operations and in this style you're writing those operations in the order in which they are performed. People come up with pretty interesting examples of these in which the sequence of operations is quite long and it is really quite natural to read the operations in the correct order. However I find that such examples do not come up often in *"real"* code. For me, the big advantage is having the important thing first and visible, as opposed to buried in amongst some function calls. This can be especially useful when you have a list of items:


```elm
let
    showInfo value =
        Html.li
            []
            [ Html.label [] [ Html.text  value ] ]
in
Html.ul
    []
    [ user.email |> String.toLower |> showInfo
    , user.name |> showInfo
    , user.dob |> calculateAge |> String.fromInt |> showInfo
    , user.points |> String.fromInt |> showInfo
    ]
```

Here I find it quite useful to see in the first column each of the things that are shown to the user. You can kind of assume that they are all shown in a reasonable or necessary way. 


## Let-in and where

In Haskell if you want a 'let' binding you have two choices, so the above, could be written as above, or using `where`:

```Haskell
Html.ul
    []
    [ user.email |> String.toLower |> showInfo
    , user.name |> showInfo
    , user.dob |> calculateAge |> String.fromInt |> showInfo
    , user.points |> String.fromInt |> showInfo
    ]
    where
    showInfo value =
        Html.li
            []
            [ Html.label [] [ Html.text  value ] ]
```

Elm of course takes the stance that it's better to keep syntax to a minimum, and there should be exactly one way to do something (if possible). So introducing a `where` clause is pretty unlikely, and I think that's a positive. One fewer decision that I have to make. Even if, perhaps in this case, *how* I show each value is probably not as interesting as which values I'm showing and hence `where` is arguably more elegant here. 

In general I have the feeling that `where` somewhat marries better with the right pizza operator. 

This is one reason why programming languages are so difficult to design, why they often become bloated. For any given feature one can always produce some example code that would be improved by its use. However, each new feature added, then means that there are various places when coding when you will have to decide which way to write some code. The fewer the features you have at your disposal the fewer ways there are to write the same bit of code. This ultimately leads to more uniform code. It also reduces cognitive strain, whilst coding you have fewer choices to make. In Elm, you might have the choice of *"should I break this expression into a let-binding or not?"* In Haskell that is already *"should I break this expression into a let-binding and if so should I use a 'let' or a 'where'?"*

There is of course one final thing to consider. Did Elm choose correctly between 'let' and 'where'? I do not know the answer to that, but my continued comfort in using the `|>` operator is beginning to make me suspect the answer is no.
