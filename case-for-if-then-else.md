---
title: "Case for if-then-else"
tags: programming elm
date: 2021-04-04
---

I mostly do not use `if-then-else`. Part of this is just a common Elm feeling of desiring to keep the language small, and since we have case expressions we do not really need `if-then-else`. But there is another part to it, I can order the branches anyway I like, I do not have to have the `True` branch first. Why would I wish to do this?

My feeling is that as you're reading code, there is only so much you can keep on your *"stack"*. So if you have a large branch of an `if-then-else` and a small trivial one, I tend to place the small trivial one first. 

This is kind of similar to defining a recursive function in which you do the base-case first, since the base-case is often quite trivial.

So for example, suppose you are rendering a sidebar menu that may or may not be open:

```elm
showSideBar : Model -> Html Msg
showSideBar model =
    if model.sideBarOpen
    then
        Html.div
            [ Attributes.id "side-bar" ]
            [ ....
            ... many lines of code
            ]
    else
        Html.nothing
```

I find if you read down to the `else` expression this way you have typically forgotten what the condition was and you have to scan/scroll back up to remind yourself what it was. There are three ways you can get around this:

### Negate the condition

```elm
showSideBar : Model -> Html Msg
showSideBar model =
    if not model.sideBarOpen
    then
        Html.nothing
    else
        Html.div
            [ Attributes.id "side-bar" ]
            [ ....
            ... many lines of code
            ]
```

### Factor out the 'then' branch


```elm
showSideBar : Model -> Html Msg
showSideBar model =
    let
        viewOpenSideBar =
            Html.div
                [ Attributes.id "side-bar" ]
                [ ....
                ... many lines of code
                ]
    in
    if model.sideBarOpen
    then
        viewOpenSideBar
    else
        Html.nothing
```

### Case and re-order


```elm
showSideBar : Model -> Html Msg
showSideBar model =
    case model.sideBarOpen of
        False ->
            Html.nothing
        True ->
            Html.div
                [ Attributes.id "side-bar" ]
                [ ....
                ... many lines of code
                ]
```

The final one of these seems more genuine to me, you're writing what you mean. Often the factoring out can also be useful, but I find that, you often need to think of two names which basically mean the same thing. Negating the condition is fine, but it's not the original question I was asking. It reads strangely, in this case it reads as "is the sidebar not open?", but it's clearly not a *bad* solution. 

I prefer the final version, it reads nicely: "Is the side bar open? If not then draw nothing, otherwise ....", and because you've got that condition off your stack you can forget about it and concentrate on the intersting code that is drawing the side bar.

So I typically use `case-of` over `if-then-else` because I can re-order the branches if I feel it reads better. It also means if I need to change the type of the condition there is less to change. A common change of type is changing between `Bool` and `Maybe`, so I might have something like:

```elm
    case mUser of
        Nothing ->
            Html.nothing
        Just user ->
            .. draw something ..
```

I then later realise that the `draw something` does not actually use the `user`, I just need to know that the user is logged in, it doesn't matter who they are logged in as, in which I might change this to:


```elm
    case Maybe.isSomething mUser of
        False ->
            Html.nothing
        True ->
            .. draw something ..
```

Or I might have started with this version and then realise I need the actual `user` value for something. Either way changing between the two is easier because I'm habitually using `case-of` instead of `if-then-else`.


There is one good reason to sometimes favour using `if-then-else`, they do not increase indentation, so if you have several nested boolean conditionals, it **might** be tidier to use `if-then-else`. Still I cannot remember the last time that happened for me.

