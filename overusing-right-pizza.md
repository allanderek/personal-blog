---
title: "Overusing Right Pizza"
tags: programming elm
date: 2021-04-05
---

Programmers are good at following rules. Rules make for reducing the number of decisions you have to make. That's why we like rules such as *"do not have functions that are longer than 300 lines of code"* much better than *"try not to have your functions too long, but sometimes you need it, so it's a judgement call"*. You might think that kind of rule gives you more flexibility, but just means you have to make more decisions. Should I really refactor this function it seems fine? That is easy to answer with the first rule, much more difficult with the second rule.

This is why when we start using something it's easy to overuse it. Only pretty recently, around a year ago, I started using the *'right pizza'* operator (`|>`) in Elm. Predictably I started overusing it.

When is the right pizza operator useful? I find for two reasons, one is chaining a bunch of operations, for example when used within the [builder-pattern](/posts/2021-01-02-builder-pattern), here is an example:

```elm
    in
    Dict.value people
        |> List.map .firstName
        |> List.filter (String.startsWith "A")
        |> List.unique
        |> List.sort
```

The right pizza operator seems correct here because it means you can write down the operations in the order in which they are performed. This is useful for being able to spot inefficiencies, for example here it is good to do the `List.unique` before the `List.sort` because that is likely to be the most expensive operation, so it is useful to do it over a shorter list, it's also clear that sorting elements you're about to remove is wasteful.

The second use case is for having the *"important"* thing first. Sometimes this is useful in a list of things, particularly things that you are rendering:

```elm
    in
    Html.div
        [ Attributes.clas "person-information" ]
        [ person.firstName |> ....
        , person.lastName |> ....
        , person.age |> ....
        , person.height |> ...
        ...
        ]
```

Here you can scan down the list of items being rendered to see *what* about a person is rendered here. The details of how each is rendered is likely less interesting, at least unless you're specifically inquiring about that, in which case it doesn't really matter what order it is written in. Even then, if for example, I wished to change how the first name is rendered, may be to make it bold, it helps that I can find *where* it is rendered quickly.

Othertimes it is just that the important thing is the *first* thing you do, as in:

```elm
update message model = 
    case message of
        UrlChanged url ->
            let
                ...
            in
            { model | route = newRoute }
                |> clearCache 
                |> initForRoute
                |> Return.noCommand
```

Here the most interesting thing about this is how the model is updated. The fact that we then run a couple of operations over the resulting model is of less importance, and it's good to show clearly that we're updating the model **first** and then, for example, initialising it for the route.

So I'm all for the right pizza operator. But it's definitely possible to start overusing it. I found myself habitually re-writing code just so that I could use the `|>` operator. A particular case is with manipulating container types, such as lists, dictionaries and sets. Because these take the container in the last place, but often the important part is the thing that you're either adding or removing.


