---
title: "Elm-format and boolean logic formulas"
tags: programming
date: 2021-03-25
---

I'm a bit surprised that I've not spoken about elm-format more on this blog. I did write [a post](/posts/2021-01-27-frugal-syntax-formatter) which mentions it, but that's more about Elm's frugal syntax and how that facilitates a formatter. The benefits of a formatter are surprisingly numerous, and many are voiced in [this elm-radio episode](https://elm-radio.com/episode/elm-format/). The main thing I like is that the trivialities of how code should be formatted are taken away from me, that's great, I can focus my attention on things that matter more. For the most part formatting decisions are just taken away from me. 

A common complaint from developers about syntax formatters, is that there are exceptions to the rules and no way to tell the formatter to just leave everything along. I can honestly say that that just does not come up much, if at all, and half the time I'm relieved to realise that I need spend no longer on the issue. I'm a pretty firm believer that I can get used to pretty much any formatting given enough time being forced to use it (similarly for pretty much any syntax). These two topics receive much more debate than is warranted.

However, I've noticed a small niggle with elm-format. Boolean logic formulas. To understand this, you have to understand that elm-format works with some small amount of guiding from the user. An expression of code can either be formatted on one line, or many lines with each node of the tree taking a new line. So you're either in 'one-line mode', or as soon as you introduce a single newline you're in multi-line mode and elm-format will insert enough new-lines as to make the expression uniform in its decision to be formatted on one line or not. This is most obvious with list expressions, you can write:

```elm
    let
        days =
            [ Mon, Tue, Wed, Thu, Fri, Sat, Sun ]
```

All on one line, and elm-format will happily keep it that way for you. However, as soon as you take a single new line, such as :

```elm
    let
        days =
            [ Mon, Tue, Wed
            , Thu, Fri, Sat, Sun ]
```
Then elm-format will re-format that into a multi-line list:


```elm
    let
        days =
            [ Mon
            , Tue
            , Wed
            , Thu
            , Fri
            , Sat
            , Sun 
            ]
```

This is great, and works for records, and function applications as well. However, my wrinkle here is with boolean logic. Sometimes you have a boolean logic something like:

```elm
let
    isJohnLennon =
        firstName == "John" && lastName == "Lennon"
```

Now if you want to have a multi-line version of this, the obvious way to format this is by having each clause on a separate line:


```elm
let
    isJohnLennon =
        firstName == "John"
        && lastName == "Lennon"
```

Unfortunately as soon as you take the new line for the `&&` you're in 'multi-line mode', and so elm-format will re-format this:


```elm
let
    isJohnLennon =
        firstName 
            == "John"
            && lastName 
            == "Lennon"
```

Hmm, that's not very readable. To fix this rule though would require a bit of work deciding on the exact rule, and I'd be worried that rule becomes very 'edgy'. You **can** kind of fix this with parentheses:


```elm
let
    isJohnLennon =
        (firstName == "John")
            (&& lastName == "Lennon")
```

It's not *quite* as nice as they I'd format it, but I can live with it.
