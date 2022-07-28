---
title: "Immutability bugs again"
tags: programming elm
date: 2021-04-07
---

Previously I've written about [immutability bugs](/posts/2021-01-23-immutabilit-bugs/) which are bugs that are more likely in an immutable language than a mutable one. I think these are relatively rare, but they do exist. A [good example](https://discourse.elm-lang.org/t/best-practice-for-updating-an-incremented-id-in-the-model/7208) has come up on the Elm discourse. 

The person asking the question wants to create new unique identifiers for items in their model. To do this you can simply keep a count of the number of identifiers you have thus far created. So you can do something like the following:

```elm
type alias Id =
    String
type alias Model =
    { ....
    , idsSoFar : Int
    }

createNewId : Model -> (Id, Model)
createNewId model =
    ( String.fromInt model.idsSoFar
        |> String.append "id-number-" 
    , { model | idsSoFar = model.idsSoFar + 1 }
    )
```

All good, however, the possibility for a bug is *relatively* high here. In your update function, if you use the `createNewId` you must make sure that you remember to store the new model. Here's a potential bug:

```elm
update : Msg -> Model -> (Model, Cmd Msg)
update message model =
    case message of
        ...
        NewThing ->
            let
                (newId, newModel) =
                    createNewId model
                newThing =
                    Thing.empty newId
            in
            ( { model | things = newThing :: model.things }
            , Cmd.none
            )
```

You see the bug, I've accidentally updated the original model rather than `newModel`. This is one reason why using static analysis tools such as `elm-review` is important. Such tools will warn you about the defined-but-unused name `newModel` and hopefully you can correct the error.

Could we find a way to make **sure** this bug doesn't happen? Yes we could, but it's not pretty. One way to do this is to define your `Model` type as an *opaque* type (this just means making it a custom tagged union type but not exporting the constructors). So, you can do the following in `Model.elm`:


```elm
module Module exposing (State, Id, Model, update, updateWithNewId)

type alias Id =
    String

type alias State a =
    { things : List Thing
    , ...
    }
type Model =
    Model (State { idsSoFar : Int} )

update : (State a -> State a) -> Model -> Model
update updateState model =
    case model of
        Model state ->
            Model (updateState state)

updateWithNewId : (Id -> State a -> State a) -> Model -> Model
updateWithNewId  updateState model =
    case model of
        Model state ->
            let
                newId =
                    String.fromInt state.idsSoFar
                        |> String.append "id-number-"
                newState =
                    { state | idsSoFar = state.idsSoFar + 1 }
            in
            Model (updateState newId newState)

```

You could also make the `Id` type opaque so that it is impossible to create one without using this module.
I think this basically solves the issue, but it's pretty far from pretty. Your update function looks like this:

```elm
update message model =
     case message of
           Tick now ->
               ( Model.update (\s -> { s | now = now }) model
               , Cmd.none
               )
           ....
           NewThing ->
              let
                    updateFun newId state =
                         { state | things = Thing.empty newId :: model.things }
              in
              ( Model.updateWithNewId updateFun model
              , Cmd.none
              )
           ...
```

You could probably make this a little more palatable by separating out your messages into those that require a new Id and those that do not and then just matching within those.
