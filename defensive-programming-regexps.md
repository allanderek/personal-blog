---
title: "Defensive programming and validations"
tags: elm programming
date: 2021-01-06
---

Defensive programming is generally defined using terms such as 'impossible', one definition on the c2 wiki has *[Defend against the impossible because the impossible will happen](https://wiki.c2.com/?DefensiveProgramming)*. In addition there is also the idea to *do the right thing*. Such a definition is obviously (and intentionally) vague. The question is, what is the *right thing*? That's always situation specific and it can be a difficult part in programming. That's what I find interesting here, the programmer can easily make happen whatever we want, but it's difficult to decide what we should want. That's difficult to follow so let's try an example.

We have an application with string inputs. Those string inputs can be validated against patterns. Because the validation happens both on the front-end (web) client (written in Elm) and the backend server code, we need a way to pass the validation patterns between the two. Now ideally we would do some code generation here so that at compile time we can check that we definitely have the validation patterns for all the string inputs that we use on the front-end. But let's assume that we cannot do that for some reason. Instead, the backend passes (in program flags) a dictionary mapping a string-input-key to a validation pattern. So when you output a string input, you might do something like:

```elm
let
    matchesWholeString s regex =
        ... 

    mPattern =
        Dict.get "email" model.inputValidationPatterns
            |> Maybe.andThen Regex.fromString


    isValidInput =
        case mPattern of
            Nothing ->
                ... WHAT NOW ..
            Just pattern ->
                matchesWholeString model.emailInput pattern

    patternError =
        case isValidInput of
            True ->
                Html.text ""
            False ->
                Html.span
                    [ Attributes.class "error" ]
                    [ Html.text "Your input is invalid" ]

    submitButton =
        case isValidInput of
            False ->
                Html.text ""
            True ->
                Html.button
                    [ Msg.SubmitEmail |> Events.onClick ]
                    [ Html.text "Submit" ]

in
Html.div
    []
    [ Html.input
        [ Attributes.value model.emailInput ]
        []
    , patternError
    , submitButton
    ]
```

So great Elm is forcing us to defensively program and deal with the *'impossible'* situation in which the key for the input type is not in the set of validation patterns provided by the backend. The question is, what should we do? What should go in the `WHAT NOW` part?

We have basically two options, we can either throw up our hands and say, okay in that case no input is valid. Or we could throw up our hands and say okay all inputs are valid (both options involve throwing up our hands).

#### Option One

```elm

    isValidInput =
        case mPattern of
            Nothing ->
                False

```


#### Option Two

```elm

    isValidInput =
        case mPattern of
            Nothing ->
                True

```

### Conflicting desires

It is useful to take a step back and decide what is desirable behaviour. In this case we would want:
1. The user to be able to submit valid input
2. The user not be able to submit invalid input
3. To be notified of bugs quickly so that we can fix them.

Using option one, the user will not be able to subit valid input, but we will surely be notified of this bug pretty quickly. Using option two, the user will be able to submit invalid input, that might be okay if we have some server side validation anyway, but it means we will not be notified about this bug soon.

### Half-way house

We can combine these two strategies. In the case that there is no validation pattern available, we can output a strange error message, but still allow for the user to submit input. This will satisfy both desires 1 and 3, but not 2:


```elm
let
    matchesWholeString s regex =
        ... 

    mPattern =
        Dict.get "email" model.inputValidationPatterns
            |> Maybe.andThen Regex.fromString


    isValidInput =
        case mPattern of
            Nothing ->
                True -- So the input is seen as valid
            Just pattern ->
                matchesWholeString model.emailInput pattern

    patternError =
        case isValidInput of
            True ->
                Html.text ""
            False ->
                Html.span
                    [ Attributes.class "error" ]
                    [ Html.text "Your input is invalid" ]

    systemError =
        case mPattern of
            Nothing ->
                Html.span
                    [ Attributes.class "system-error" ]
                    [ Html.text "There is an internal error relating ..." ]

    submitButton =
        case isValidInput of
            False ->
                Html.text ""
            True ->
                Html.button
                    [ Msg.SubmitEmail |> Events.onClick ]
                    [ Html.text "Submit" ]

in
Html.div
    []
    [ Html.input
        [ Attributes.value model.emailInput ]
        []
    , patternError
    , submitButton
    , systemError
    ]
```

Obviously it depends on the particular situation, but by writing down the desires we can arrive at the least bad solution. In this case we were more interested in allowing valid input, rather than in disallowing invalid input, other situations may of course be different. The point here is that when accounting for impossible situations the *'correct'* thing to do is not always obvious. Often being alerted to the problem is a strongly desired attribute.

