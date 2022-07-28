---
title: "Or patterns"
tags: elm syntax 
date: 2021-01-07
---

Elm is famously pretty conservative when it comes to adding language features/syntax. In general I appreciate this, but or-patterns are something I have longed for for a long time. I feel they can drastically increase clarity. However I do understand that a *'better'* solution may arrive and we will all be glad Elm held off on implementing or-patterns. Still I feel that in this case, it's worth the risk, or-patterns are used in a few other languages and to good effect.

## What are or-patterns

A simple implementation would simply allow you to 'stack' case branches together, here is a simple example:

```elm
isWeekEnd : Day -> Bool
isWeekEnd day =
    case day of
        Sat ->
        | Sun ->
            True
        Mon ->
        | Tue ->
        | Wed ->
        | Thur ->
        | Fri ->
            False
```

You can get progressively more fancy with this. You can allow the patterns to define variables as long as all of the patterns which are stacked together define the same set of variables at the same types:

```elm
type alias Age = String
type Actor
    = Person Age
    | Animal Species Age

actorAge : Actor -> Age
actorAge actor =
    case actor of
        Person age ->
        | Animal _ age ->
            age
```

Next you can allow such or-patterns nested within other patterns:

```elm
type alias Role = (Name, Actor)
roleAge : Role -> Age
roleAge role =
    case role of
        (_, Person age | Animal _ age) ->
            age
```

Finally you could allow missing variables to be defined in the pattern, though personally I find this is probably going further than is warranted, I don't think I've ever really wanted this:

```elm
nothingAsZero : Maybe number -> number
nothingAsZero mNumber =
    case mNumber of
        Nothing n is 0 ->
        | Just n ->
            n

```

Of course these silly little examples do not really show off the power of or-patterns. They really come into their own when defining utility functions over custom types with many cases. For example, your `Message` type, something like:

```elm
isUserInteraction : Message -> Bool
isUserInteraction message =
    case message of
        EmailInput _ ->
        | PasswordInput _ ->
        | LoginSubmit ->
        ... bunch of other cases ..
            True
        LoginResponse _ ->
        | ProfileResponse _ ->
        | PostsResponse _ _ ->
        ... bunch of other cases ..
            False

```

## Why might something better come along?

Or-patterns do not solve **all** the issues with case expressions. In particular some times you need a sub-expression for some of the cases, but those cases do not have identical results. This kind of situation is awkward to describe and doesn't lend itself to short self-contained examples very well. 



## Conclusion

I still think or-patterns should be introduced into Elm, or pretty much any other functional language that does not have them. I don't believe they hurt comprehension, or make the compiler particularly more complex, or make error messages more difficult (though parsing might be an issue here).
I wrote a more thought out proposal a couple of years ago, and I still think it stands well: https://discourse.elm-lang.org/t/language-proposal-introduce-or-patterns/2664/7, the thread is also worth reading, some good contributions in there.
