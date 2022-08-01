---
title: "Dry can cost you"
tags: programming
date: 2021-04-08
---

I had an issue with a computation taking much longer than might have been expected. It turned out to be caused by a use of the function `greedyGroupsOf` from the [list-extra library](https://package.elm-lang.org/packages/elm-community/list-extra/latest/List-Extra), and it was solved by writing my own, naive, implementation.

I haven't quite gotten to the bottom of **why** the library version of `greedyGroupsOf` of is so much slower than my naive version, but the reason it is slower is mostly to do with *DRY* or *don't repeat yourself*.

To see why, let's look first at my naive version:

```elm
greedyGroupsOf : Int -> List a -> List (List a)
greedyGroupsOf size elements =
    case elements of
        [] ->
            []

        _ ->
            let
                ( year, remainder ) =
                    List.splitAt size elements
            in
            year :: splitIntoYears size remainder
```

This is perfectly simple. Now, let's look at the [library version](https://github.com/elm-community/list-extra/blob/8.3.0/src/List/Extra.elm#L1927):

```elm
greedyGroupsOf : Int -> List a -> List (List a)
greedyGroupsOf size xs =
    greedyGroupsOfWithStep size size xs
```

Ah, okay, so it's just calling a more general function. This is why it is *DRY*, I do this kind of thing, almost habitually. Here is the implementation of the [more general function](https://github.com/elm-community/list-extra/blob/8.3.0/src/List/Extra.elm#L1951):

```elm
greedyGroupsOfWithStep : Int -> Int -> List a -> List (List a)
greedyGroupsOfWithStep size step xs =
    let
        xs_ =
            List.drop step xs

        okayArgs =
            size > 0 && step > 0

        okayXs =
            List.length xs > 0
    in
    if okayArgs && okayXs then
        List.take size xs :: greedyGroupsOfWithStep size step xs_

    else
        []
```

As I've mentioned I'm not entirely sure why this is so much slower than the more specific function. There is one thing it does. Because the step and the size might be different it cannot use `List.splitAt` and has to do two separate `List.drop` and `List.take` operations. This is because in the more general function you can have a different step size to that of the group size, a lower step size will mean that some elements are in more than one group, whilst a larger step size will mean some elements are omitted entirely.

Anyway, this means that `greedyGroupsOf` is slower than it needs to be, because it is a simple invocation of the more general function. In this case, it might be worth just using the specialised implementation of the function. It means repeating some code, or logic, but in this case I doubt that's so bad, these implementations are unlikely to change, and they are probably correct.
