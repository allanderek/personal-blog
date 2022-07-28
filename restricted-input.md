---
title: "Restricted text input in Elm"
tags: Elm programming
date: 2021-03-02
---

A little tip for restricted text input in Elm. I've coded up the general idea in an [ellie](https://ellie-app.com/cwm5rQdMChGa1). This is really a general HTML tip, that I didn't know existed, but for some reason I feel it works pretty well in Elm.

The sitation is that you wish to restrict a text input to one of several possibilities, but there are many possibilities, so a `select` element is probably not correct. You may or may not wish to prevent any output not in the list of suggestions. What I didn't know is that you can define a `datalist` element that has all the suggestions and the browser will interpret that appropriately provided you set the `list` attribute on the `input` element. As for invalidating it, you could use a `pattern` attribute, but I think in Elm you're probably doing your own invalidation anyway.

So, supposing you have an input, and some list of appropriate inputs - maybe they are dynamic, or a list of countries or something - the key point is to define the `datalist` element:

```elm
import Html exposing (Html)
import Html.Attributes as Attributes

allowableDataList : String -> List String -> Html msg
allowableDataList id allowableInputs =
    let
        makeOption input =
            Html.option
                [ Attributes.value input
                , Attributes.id id 
                ]
                []
    in
    List.map makeOption allowableInputs
        |> Html.datalist []
```

Now, you might wish to define this in your main `view` function if, for example, it's not often changing and used frequently. But you can always define it when you need it. Of course you have to be careful with the value of the `id`.  To use this, you only have to set the `list` attribute on an input:

```elm
import Html exposing (Html)
import Html.Attributes as Attributes
import Html.Events as Events

restrictedInput : String -> (String -> Msg) -> String -> Html Msg
restrictedInput currentInput onInput allowInputsId =
    Html.input
        [ Attributes.value currentInput
        , Attributes.list allowInputsId
        , Events.onInput onInput
        ]
        [] 
```

The key point here is `Attributes.list allowInputsId`. As I said above you could define the `datalist` in your main function, or you could also define it here very locally, even within the `input` works:


```elm
import Html exposing (Html)
import Html.Attributes as Attributes
import Html.Events as Events

type alias RestrictedInput =
    { onInput : String -> Msg
    , id : String
    , allowableInputs : List String
    , currentValue : String
    }

restrictedInput : RestrictedInput -> Html Msg
restrictedInput config =
    let
        dataListId =
            String.append config.id "-data-list"
    in
    Html.input
        [ Attributes.value config.currentInput
        , Attributes.id config.id
        , Attributes.list dataListId
        , Events.onInput onInput
        ]
        [ allowableDataList dataListId config.allowableInputs ]
```

Then if you also want to display an error message in the case that the current input is not one of the allowable inputs then you can simply check `List.member config.currentInput config.allowableInputs`. Obviously you might wish to augment that with whether the input has been *blurred*. Anyway I thought this was a nice little tip worth sharing, one of those little not-well-known corners of HTML. As I said before there is an [ellie-app](https://ellie-app.com/cwm5rQdMChGa1) for this.
