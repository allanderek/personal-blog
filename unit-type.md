---
title: "Unit type and empty records"
tags: elm syntax
date: 2021-02-04
---

An [interesting thread](https://discourse.elm-lang.org/t/unit-type-purpose/6848) came up on the [Elm discourse](https://discourse.elm-lang.org/) today. It concerned the point of the `unit` type `()` (which is also the only value of type unit). The question was do we really need this? Couldn't we use the empty record `{}`. This then provoked the question of whether we even need tuple types at all, why not just always insist on record types?

I find it quite interesting that even in Elm a language that is far more frugal with its syntax than most, you can still find parts of the grammar that have questionable use. I think getting rid of both the unit type and more generally tuples would be perfectly doable, though I doubt it will happen since it would be somewhat inconvenient for most since tuples are even used as the results of both the `init` and `update` functions. Hence **all** programs and probably a large proportion of libraries would need to be updated for the change. Nonetheless it's pretty fun to imagine the language without tuple types.

Firstly, the `()` type, clearly in any case we use this we could instead use `{}` the empty record type (and value). It doesn't seem useful to have two ways to do the same thing. The empty record type is most often used as the parameter to an extensible record type when you just do not want to add any additional fields:

```elm
type alias Field a =
  {a | field : Bool }

type alias MyRecord =
  Field {}
```

It turns out, as we found out in the thread linked above, that you can also use the unit type for that purpose. Still if we're getting rid of one or the other I would vote get rid of the unit type.

I've always found it to be something of an anomoly (not just in Elm but most functional languages) that you have this progression:
* `()` a tuple of length zero (or the unit value).
* `(1)` not a tuple at all
* `(1,2)` a tuple of length two.
* `(1,2,3)` a tuple of length three.

So you cannot have a tuple of length one, even though you can have one of length zero. This of course does not hold for records, which you can with 0, 1, 2, 3, ... fields, up to some maximum I suppose.

So, again you have the fact that parentheses are used for two distinct things, to surround an expression, and also to create a tuple. I think in O'caml the parentheses are not formally part of tuples, it is the comma that creates the tuple and the fact that the programmer regularly surrounds a tuple expression in parentheses is not enforced.

Anyway the main question is, can we get rid of tuples. I personally would not grieve to see them go. However, you would need a solution for case expressions that match against multiple values such:

```elm
maybeContains : Maybe String -> Maybe String -> Bool
maybeContains mLeft mRight =
    case (mLeft, mRight) of
        ( Just left, Just right ) ->
            String.contains left right
        _ ->
            False
```
Though you could just allow multiple expressions and patterns separated by commas in exactly this position, rather than tuples more generally. Alternatively if you could pattern match on record values such as:


```elm
maybeContains : Maybe String -> Maybe String -> Bool
maybeContains mLeft mRight =
    case {left = mLeft, right = mRight} of
        { left = Just l, right = Just r } ->
            String.contains l r
        _ ->
            False
```

I certainly find that less appealing and I had to think up two extra names, because `{ left = Just left ...` is probably not a good idea. I confess that I occasionally find myself wishing to be able to pattern match against a record value in such a way as `{ field-name = pattern }`. 


Note that if you just allow multiple expressions separated by commas within case expressions, then you won't be able to directly pattern match against the result of a function. So I think allowing patterns in records would be the way to allow for this, but I still find it a touch uglier.

All-in-all, I could probably survive without tuples, particularly if records could be pattern matched against. But I would have no qualms at all about getting rid of the unit type.
