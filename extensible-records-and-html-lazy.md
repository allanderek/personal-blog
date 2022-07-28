---
title: "Html.Lazy and extensible records"
tags: elm programming
date: 2021-03-07
---

I really like using Elm's extensible records. There is a little debate about how best to utilise them. It seems clear they should be used for narrowing the types of function arguments. Narrowing the type of an argument to a function often makes it significantly more general. Here's a quick example, suppose you have a `User` type in your application, you might have only a few of them, say a list of friends of the current user or something. So you might write a function to find a particular user in a list:

```elm
type alias UserId = String
type alias User =
    { id : UserId
    , name : String
    , ...
    }

findUser : UserId -> List User -> Maybe User
findUser userId users =
    List.find (\u -> u.id == userId) users
```

All great, but often the `id` part of an entity comes from the database, and we actually have many such entities in our application, so we can actually make the `findUser` function much more general by using an extensible record type for the entity type:


```elm
findEntity : comparable -> List { a | id : comparable } -> Maybe { a | id : comparable }
findEntity id entities =
    List.find (\e -> e.id == id) entities
```

Now this works for all of the entities in your program. The second use of extensible record types is to actually model data. At one point Evan suggested they should not be used for that, so you tend to get a bit of pushback against this idea, but clearly extensible records are useful for **some** data modelling issues.

Anyway, what I really wanted to talk about today was a small conflict between the use of extensible records and `Html.lazy`. The idea behind `Html.lazy` is a very good one. Often in your Elm application you will render some complicated, or expensive, element, perhaps it is a list of something, say products in an e-commerce store, or footballers in a fantasy football application. Many messages which invoke the `update` function will not change the list of products/footballers etc. So it is a shame to re-render this list on every `view`. That's where `Html.lazy` comes in, if you use `Html.lazy` instead, then Elm's runtime *memoizes* the rendered list and only re-renders it, if the inputs to the renderer changes. Let's make this a bit more concrete with some code, let's suppose we're authoring a fantasy football application:

```elm
type alias Model =
    { players : List Player
    , filters : Filters
    , entry : Entry
    , now : Time.Posix
    , ...
    }

renderPlayers : Filters -> List Player -> Html Msg
renderPlayers filters players =
    ...

view : Model -> Html Msg
view model =
    Html.div
        []
        [ Html.header [] [ ... ]
        , ...
        , Html.lazy2 renderPlayers model.filters model.players
        , ...
        , Html.footer [] [ ... ]
        ]
```

An alternative to this style for `renderPlayers` is just to take the entire model in, but use extensible records to only record the parts that you care about:


```diff
type alias Model =
    { players : List Player
    , filters : Filters
    , entry : Entry
    , now : Time.Posix
    , ...
    }

- renderPlayers : Filters -> List Player -> Html Msg
- renderPlayers filters players =
+ renderPlayers : { a | filters : Filters, players : Players } -> Html Msg
+ renderPlayers { filters, players } =
    ...

view : Model -> Html Msg
view model =
    Html.div
        []
        [ Html.header [] [ ... ]
        , ...
-        , Html.lazy2 renderPlayers model.filters model.players
+        , Html.lazy renderPlayers model 
        , ...
        , Html.footer [] [ ... ]
        ]
```

However, there is a problem here. The `Html.lazy` is essentially useless, because any update that changes the `model` at all, will cause the `renderPlayers` to be re-run, even though neither the `filters` or the `players` in the `model` have been changed.

It's possible that the Elm compiler and run-time could be updated to make this *just work*, but currently this is a trade-off that you need to take into consideration when deciding how to model your data and construct your views.
