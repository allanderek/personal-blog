---
title: Underscore type params
description: A very small feature proposal for Elm's type parameters with a motivating situation.
date: 2022-07-26
---

In a case expression when you want to match **anything** you can use the special pattern `_` which avoids giving a name to something you aren't going to use. However, you cannot do the same thing for type parameters. If you want to say "I accept any type here and I don't return anything containing that type" you just have to use an unused ttype variable name. I think you should be allowed to "say what you mean" and use underscore here for that situation. I'm going to give a motivating circumstance in which this would actually be useful.

To give a concrete example, suppose you wish to define the `length` function for lists:

```elm
length : List a -> Int
length l =
    case l of
        [] -> 0
        _ :: rest -> 1 + length rest
```

I think you should be able to write the type signature here as `List _ -> Int`. Note that this only works because we're not returning any of the elements, we could **not** do: `reverse : List _ ->  List _`. In just the same what that you cannot use `_` as a variable name. So it is crucial that underscore does not simply introduce a new type variable name into scope which must be unified. 

A particularly common place where you don't care about type parameter is when used within an extensible record. Suppose you have the following function:

```elm
viewSideBar : { a | now : Time} -> User -> Html msg
viewSideBar model =
    let
        showTime : Html msg
        ...
        showUser : Html msg
        ...

    in
    Html.div
        [ Attributes.class "sidebar" ]
        [ showTime 
        , showUser 
        ]

```

Now suppose the user may have a list of pets, as well as a list of siblings, ignoring the possibility that either of these two lists is empty:

```elm
viewSideBar : { a | now : Time} -> User -> Html msg
viewSideBar model =
    let
        showTime : Html msg
        ...
        showUser : Html msg
        ...

        showPets : Html msg
        showPets =
            Html.ul
                [ Attributes.class "user-pets" ]
                [ List.map (\p -> Html.li [] [ Html.text p.name]) user.pets  ]

        showSiblings : Html msg 
        showSiblings =
            Html.ul
                [ Attributes.class "user-siblings" ]
                [ List.map (\s -> Html.li [] [ Html.text s.name]) user.siblings  ]

    in
    Html.div
        [ Attributes.class "sidebar" ]
        [ showTime 
        , showUser 
        , Html.h2 [] [ Html.text "Pets" ]
        , showPets
        , Html.h2 [] [ Html.text "Siblings" ]
        , showSiblings
        ]

```

Now, a `Pet` is a different type from a `Sibling` but you realise that both are rendered the same, as both have a `name` field and that's all that is used here, so you factor that out:

```elm
viewSideBar : { a | now : Time} -> User -> Html msg
viewSideBar model =
    let
        showTime : Html msg
        ...
        showUser : Html msg
        ...

        showNamedList : String -> List { a | name : String } -> Html list
        showNamedList className nameables =
            Html.ul
                [ Attributes.class className ]
                [ List.map (\n -> Html.li [] [ Html.text n.name]) nameables  ]


        showPets : Html msg
        showPets =
            showNamedList "user-pets" user.pets

        showSiblings : Html msg 
        showSiblings =
            showNamedList "user-siblings" user.siblings

    in
    Html.div
        [ Attributes.class "sidebar" ]
        [ showTime 
        , showUser 
        , Html.h2 [] [ Html.text "Pets" ]
        , showPets
        , Html.h2 [] [ Html.text "Siblings" ]
        , showSiblings
        ]

```

Can you spot the error? I've re-used the `a` as a type parameter, but that now means the type checker will insist that the uses of `showNamedList` are unified to the type in the main signature. It won't be able to do that because presumably the other fields of `Pet`s and `Sibling`s are different. Even if they are the same the `{a | now : Time}` would now be too lenient.  

What you really  intended here was `{ anything | now : Time}`, and I think a reasonable way to write that is `{ _ | now : Time}`.
