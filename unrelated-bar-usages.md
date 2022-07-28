---
title: "Unrelated bar usages"
tags: elm syntax
date: 2021-02-27
---

This post is pure, unadulterated bike-shedding, on the use of the `|` operator for extending a record type.

As I've said many times before, one of the big draws of Elm is the clean, and very slim syntax. This has numerous benefits. One thing I never noticed before but has just been [pointed out on the Elm discouse](https://discourse.elm-lang.org/t/pipe-operator-in-extensible-record-type/7012) is that the *bar operator* `|` is used for two different, unrelated record-type meanings in Elm. The first is within a record *type* to indicate that the record is an extensible one:

```elm
type alias Pointful a =
    { a | x : Int, y : Int }
```

Note it is not just type *definitions*, but anywhere a 'type' is valid syntax:

```elm
viewUser : { a | user : Maybe User, route : Route } -> Html Msg
viewUser model =
    ...
```

The second use of the bar-operator is in a record **expression** to indicate that  you want a new record constructed by copying across all the non-mentioned fields of the given record:

```elm
moveRight : Int -> Point -> Point
moveRight delta point =
    { point | x = delta + x }
```

It does seem a little strange to me that the operator is doubled up like this. It is of course reasonable to double up an operator within a 'type' and an 'expression', however it is less of a surprise if they mean analogous things. For example `->` is used in both function types and lambda expressions, though also case expressions. Again the doubling up of `->` in lambda and case expressions is basically a similar if not entirely the same use. Certainly the `->` means an analogous thing in types and in lambda expressions.

Anyway for `|` it's not clear how this came about, or whether it's a particular problem. The good thing is that doubling this up doesn't seem to have caused me, or most other Elm programmers any bother and that means there are operators available for other things should they be needed. I suppose a slightly more consistent re-used operator might have been `::`. Because `x :: l` means *'extend the list `l` at the front by adding the value `x`'*, so if we used it for record extension it would be kind of appropriate. This would likely have the issue that you might easily mis-type `:` for `::` or vice-versa.

I suppose `|` is also used to *'extend'* custom taggged types. Another possibility would just have been to use a keyword, say `extends`. All this goes to show that no matter how clean and unobjectionable your syntax, it can still e bike-shedded all day long.
