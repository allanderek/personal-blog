---
title: "Hungarian notation and Elm maybes."
tags: elm syntax 
date: 2021-01-21
---

There is an old but excellent [post from Joel on Software](https://www.joelonsoftware.com/2005/05/11/making-wrong-code-look-wrong/) which defends hungarian notation. Briefly hungarian notation was mis-applied in many places, and so something that wasn't nearly as useful as hungarian notation was adopted and then (rightly) despised. That post is great and worth reading if you haven't, but it is also quite long, so I'm going to first of all explain the correct hungarian notation that is useful, then the incorrect despised hungarian notation.  Finally, I'm going to end by discussing how the old despised hungarian notation relates to Elm maybes.

## Correct hungarian notation

The idea is that some types are the same, but refer to different *kinds* of things. The standard example was a length specified in particular units, which might be, cm, px, or ch (centimeters, pixels, or characters). So the idea with hungarian notation is, if you have a variable name that represents a length, prefix the name with a prefix that specifies the units. So you might have:

```elm
let
    pxScreenWidth = pxGetScreenWidth(..)
in
max pxScreenWidth player.pxHorizontalPosition
```

The idea is, that if you make a mistake, this should be noticable to anyone reading the code:


```elm
let
    pxScreenWidth = pxGetScreenWidth(..)
in
max pxScreenWidth player.cmHorizontalPosition
```

Just by looking at this, you know there is some mistake in the code. It might be that `pxGetScreenWidth` does actually return the screen width in centimeters, but then it is mis-named. Or it could be that `player.cmHorizontalPosition` does actually hold the horizontal position in pixels, but then **that** is mis-named. More likely, you're trying to combine two measurements in different units and that just doesn't make sense to do.

Now, of course, with a type system a bit more expressive than C's, you could get the compiler to help you out here. You can create new types for lengths in each of the three (or however many there are) units of measurements, and then you wouldn't be able to mix and match. That's one reason you don't tend to see hungarian notation in Elm code. However, another reason is that hungarian notation got warped, and disliked.

## Incorrect hungarian notation

Unfortunately, there was a mistake in the use of hungarian notation, in which instead of the 'kind' of the variable in question, the literal type of the variable was written as the prefix. So the above would become:


```elm
let
    intScreenWidth = intGetScreenWidth(..)
in
max intScreenWidth player.intHorizontalPosition
```

Which of course is utterly useless in helping you to spot coding errors. If you're wondering how anyone could possibly make such a mistake as this, remember that this was popularised in C, where you have a few different types that all represent an integer: `char, unsigned char, short, unsigned short, int, unsigned int, long, unsigned long`, at least. So I guess there was some idea that this had some use. Still, this form of hungarian notation was mostly just baggage, which didn't really help you spot mistakes, and in fact made a bunch of variable names look similar, and also made it difficult to change the type of a variable. It was rightly disliked and fell out of fashion.

Joel (on Software) put this mistake down to Simonyi's use of the word *type* (rather than say *kind*) and popular Windows programming books like Charles Petzold's. 

## Hungarian Notation and Elm Maybes

Okay, so what does this have to do with `Maybe` types in Elm? I find myself sort of using this style of naming variables, but for a different reason. I regularly prefix my variable name with an `m` if it is a `Maybe` type. So I might do something like the following:

```elm
showUser : Maybe User -> Html msg
showUser mUser =
    ...
```

The compiler of course will prevent me from using the value `mUser` as a straight-up `User` type, so why would I bother? Well, it's about finding names, to implement my `showUser` function, I probably have to pattern match on the `Maybe` and then find a name for the resulting `Just` argument:


```elm
showUser : Maybe User -> Html msg
showUser mUser =
    case mUser of
        Nothing ->
            Html.text "Anonymous"
        Just user ->
            Html.span
                []
                [ Html.text user.name
                , Html.text " ("
                , Html.text user.email
                , Html.text ")"
                ]
```

Had I named the original `mUser` just `user` I'd be stuck for a decent name for `Just ...`.

