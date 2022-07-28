---
title: "Stale Messages in Elm"
tags: elm programming
date: 2021-02-16
---

Coury Ditch reported on [the trickiest Elm bug](https://blog.realkinetic.com/the-trickiest-elm-bug-ive-ever-seen-988aff6cbbd7) he's ever seen. It's a good read and a good caution against holding stateful information into your Elm messages. When I was very new to Elm one of the first things I had to do at a new company was figure out a bug that basically had the *'Stale message'* anti-pattern as main reason behind it. So I'm going to described the bug and the solution.

The company's flagship product included a registration form, but being a financial application the registration form was a bit more onerous than simple email and password. It had around eight fields or so. The problem was that Chrome's autofill feature wasn't working, it was filling in the last field only. Why?

So, you can think of the registration form, although it had at least 8 fields, as something like this:

```elm
type alias RegForm =
    { firstName : String
    , lastName : String
    , email : String
    , password : String
    , passwordRepeat : String
    }
```

This was just stored at the top level in the main `Model` record type.  Now, whoever had initially developed the app had thought it would be a neat shortcut to avoid creating a message for each input field, and just create one message, with the new registration form:

```elm
type Msg
    = ...
    | UpdateRegForm RegistrationForm
    ...

type alias Model =
    { regForm : RegistrationForm
    , ...
    }

update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        UpdateRegForm newForm ->
            ( { model | regForm = newForm }
            , Cmd.none
            )
        ...

```

Now when viewing a field of the registration form, the entire form was updated.  So for example the `firstName` field might have been something like:

```elm
import Html exposing (Html)
import Html.Attributes as Attributes
import Html.Events as Events


viewRegForm : { a | regForm : RegistrationForm } -> Html Msg
viewRegForm { regForm } =
    let
        firstName =
            Html.div
                [ Attributes.class "first-name-field" ]
                [ Html.label [] [ Html.text "First name" ]
                , Html.input
                    [ Events.onInput (\input -> { regForm | firstName = input })
                    , Attributes.value regForm.firstName
                    ]
                    []
                ]

    ...
```

In reality it was a little more complicated and I think there was a helper function to draw a text field input. The main point however is the argument to the `Events.onInput`, this, as always, is a function that takes the new value of the input field. The difference here is that the result of the function is the entire registration form that is used to replace the existing on on the model in the `update` function.

The problem was, that because Chrome was filling out several fields, essentially simultaneously (because they were all in the same animation frame), then each update of a field over-wrote the previous update. In other words you got a bunch of messages such as:

```elm
UpdateRegForm { currentRegForm | firstName = "Billy" }
UpdateRegForm { currentRegForm | lastName = "Shears" }
UpdateRegForm { currentRegForm | email = "billy@sergeantpeppers.com" }
UpdateRegForm { currentRegForm | password = "1234" }
UpdateRegForm { currentRegForm | passwordRepeat = "1234" }
```

This is because each update message is calculated from the same model, and because in the `update` function you're replacing the entire registration form, all the previous updates are lost. 

The proper solution would have been to suck it up and create a message for each field, as in:

```elm
type Msg
    = ...
    | UpdateRegFormFirstName String
    | UpdateRegFormLastName String
    ...
```

That way because each message only contains the information for its own associated field, it only updates that part of the registration form, and hence subsequent ones, even if done in the same animation frame (ie. without an Elm render in between) will not overwrite previous messages' changes:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        UpdateRegFormFirstName input ->
            let
                oldForm =
                    model.regForm
                newForm =
                    { oldForm | firstName = firstName }
            in
            ( { model | regForm = newForm }
            , Cmd.none
            )
        ...
```

Autofill now works perfectly.  However, if you like the idea of a single message to update the registration form, you can still have that. You just need to put the **function** in the message. The result is that your message is still saying how to update the registration form, rather than what to update the registration form with:

```elm
type Msg
    = ...
    | UpdateRegForm (RegistrationForm -> RegistrationForm)
    ...

type alias Model =
    { regForm : RegistrationForm
    , ...
    }

update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        UpdateRegForm transformRegForm ->
            ( { model | regForm = transformRegForm model.regForm }
            , Cmd.none
            )
        ...
```

Then in the view function you just have to slightly update the `onInput` attribute:


```elm
...
        firstName =
            ...
                , Html.input
                    [ Events.onInput (\input oldRegForm -> { oldRegForm | firstName = input })
                    ...
    ...
```

The disadvantage is that now you have a function in your message type. I'm not sure if the debugger still has a problem with that, but it is generally seen as desirable to keep functions out of your message type.

The main point here though is that you need to try to avoid having in your message type parts of the model. Because the model may have changed since the view was rendered and hence the message constructed.
