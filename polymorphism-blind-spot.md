---
title: "Polymorphism blind spot"
tags: programming elm
date: 2021-04-01
---

I am a huge fan of polymorphism and of accompanying static type checking. I find that, meta-programming aside, I mostly do not wish to write the kinds of things that are allowed by dynamic typing but disallowed by static typing. There are two exceptions, or blind spots to this and I thought I would detail the first one. 

Suppose you have a data structure with several `Maybe` fields:

```elm
type alias Preferences =
    { favouriteFood : Maybe Food
    , favouriteBand : Maybe Band
    , favouriteColor : Maybe Color
    , favouriteNumber : Maybe Int
    , favouriteName : Maybe String
    }
```

Now suppose you want to count how many of these are set:

```elm
numSetPreferences : Preferences -> Int
numSetPreferences preferences =
    [ preferences.favouriteFood
    , preferences.favouriteBand
    , preferences.favouriteColor
    , preferences.favouriteNumber
    , preferences.favouriteName
    ]
        |> List.count Maybe.isSomething
```

This is nice, but will not compile. The problem is that although the function `Maybe.isSomething` has the type `Maybe a -> Bool` so we don't really care what are in the `Maybe` types of the list, you cannot write down the type of the list.

In this case you can get out of this, although it's definitely not as elegant:


```elm
numSetPreferences : Preferences -> Int
numSetPreferences preferences =
    [ preferences.favouriteFood |> Maybe.isSomething
    , preferences.favouriteBand |> Maybe.isSomething
    , preferences.favouriteColor |> Maybe.isSomething
    , preferences.favouriteNumber |> Maybe.isSomething
    , preferences.favouriteName |> Maybe.isSomething
    ]
        |> List.count identity
```

This is a fairly minor irritation and I'm more than willing to live with this.
