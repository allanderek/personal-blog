---
title: "Generating from types"
tags: programming elm
date: 2021-02-08
---

I actually managed to do a little bit of work on the pole-prediction backend last night. There is a part where we store messages in the *"database"* (it's not really a database, it's really just persistent storage). So the messages are a variant type, and as such we need to write code to both encode each message into JSON, and decode each message from JSON.

```elm
type DatabaseMessage
    = AddDriver Year Driver
    | AddTeam Year Team
    | AddEntrant Driver.Id Team


encodeDatabaseMessage : DatabaseMessage -> Encode.Value
encodeDatabaseMessage dMsg =
    case dMsg of
        AddDriver year driver ->
            [ ( "tag", "AddDriver" |> Encode.string )
            , ( "arg1", year |> Encode.int )
            , ( "arg2", driver |> Driver.encode )
            ]
                |> Encode.object

        AddTeam year team ->
            [ ( "tag", "AddTeam" |> Encode.string )
            , ( "arg1", year |> Encode.int )
            , ( "arg2", team |> Team.encode )
            ]
                |> Encode.object

        AddEntrant year driverId team ->
            [ ( "tag", "AddEntant" |> Encode.string )
            , ( "arg1", driverId |> Encode.string )
            , ( "arg2", team |> Team.encode )
            ]
                |> Encode.object


databaseMessageDecoder : Decoder DatabaseMessage
databaseMessageDecoder =
    let
        interpret s =
            case s of
                "AddDriver" ->
                    Decode.succeed AddDriver
                        |> Decode.andField "arg1" Decode.int
                        |> Decode.andField "arg2" Driver.decoder

                "AddTeam" ->
                    Decode.succeed AddTeam
                        |> Decode.andField "arg1" Decode.int
                        |> Decode.andField "arg2" Team.decoder

                "AddEntrant" ->
                    Decode.succeed AddEntrant
                        |> Decode.andField "arg1" Decode.string
                        |> Decode.andField "arg2" Decode.string

                _ ->
                    Decode.fail (String.append "Unknown message string: " s)
    in
    Decode.field "tag" Decode.string
        |> Decode.andThen interpret
```

As you can see all of this is very repetitive and lends itself well to being automatically generated. You can easily imagine some meta-code that, given a type definition, can automatically generate an encoder and decoder (there is also the [elm-codec library](https://package.elm-lang.org/packages/miniBill/elm-codec/latest/) but if you're auto-generating these anyway then that's less useful).

Anyway, imagining some type for the abstract syntax of Elm code both of these functions could be generated, even from the parsed type definition. Which is great since it means that there isn't any extra burden on the programmer of writing your type definition in some meta code. That is, it's much nicer to write:

```elm
type DatabaseMessage
    = AddDriver Year Driver
```

than

```elm
databaseMessageType : ElmSyntax.TypeDef
databaseMessageType =
    { name = "DatabaseMessage"
    , constructors = 
        [ { name = "AddDriver"
          , args = [ Name "Year", Name "Driver" ]
          }
        ]
    }
```

Using some imagined meta-programming library for Elm. However, there is a small niggle here. At some point, you may wish to update the type definition in a backwards compatible way. Adding a constructor is fine, you can still just generate code from the type definition, the fact that there is *additional* code being generated is fine. However, suppose I wish to **change** the `AddDriver` constructor. Suppose it now also wants a driver number. So we now want:


```elm
type DatabaseMessage
    = AddDriver Year Driver Int
```

The problem is that the obvious generated code for the encoder and decoders will assume that all messages in the **existing** store can be decoded as having a third parameter, when they cannot. Now it's easy enough to make this backwards compatible, You can just handle the case that there is no `arg3`, so instead of the following:

```elm
                "AddDriver" ->
                    Decode.succeed AddDriver
                        |> Decode.andField "arg1" Decode.int
                        |> Decode.andField "arg2" Driver.decoder
                        |> Decode.andField "arg3" Decode.int
```

you can instead write:

```elm
                "AddDriver" ->
                    Decode.succeed AddDriver
                        |> Decode.andField "arg1" Decode.int
                        |> Decode.andField "arg2" Driver.decoder
                        |> Decode.optionalField "arg3" 0 Decode.int
```

The question is, how do we communicate that to the meta-programming system if we're just writing the normal Elm type definition? There are a few options:

* Force people to never change constructors, you can just use a new constructor name and then factor out the common update stuff, for example you could do:

```elm
type DatabaseMessage
    = AddDriver Year Driver -- Doesn't change
    | AddDriverWithNumber Year Driver Number


... in an update function ...
    AddDriver year driver ->
        update (AddDriverWithNumber year driver 0) model
```

* Always generate code that just picks a default for when the argument isn't there.

```elm
                "AddDriver" ->
                    Decode.succeed AddDriver
                        |> Decode.optionalField "arg1" 0 Decode.int
                        |> Decode.optionalField "arg2" Driver.empty Driver.decoder
                        |> Decode.optionalField "arg3" 0 Decode.int
```

* Force migrations

You can even generate the migrations given two type definitions.

* Write the type definition in the ugly meta-programming library syntax, but have someway to specify that an argument to a constructor maybe missing at decode time.

```elm
databaseMessageType : ElmSyntax.TypeDef
databaseMessageType =
    { name = "DatabaseMessage"
    , constructors = 
        [ { name = "AddDriver"
          , args = 
            [ { kind = Name "Year", decoder = strictStringField }
            , { kind = Name "Driver", decoder = strictDriverField }
            , { kind = Name "Number", decoder = optionalIntField 0 }
            }
        ]
    }
```

* Introduce some commenting on type definitions to specify this, such as:

```elm
type DatabaseMessage
    = AddDriver Year Driver {- optional -} Int
```

Probably some other solutions I haven't thought of. All-in-all this is a non-trivial problem to solve, which is why so far I haven't done any meta-programming here at all, and just written all the encode/decoders for messages by hand. Sometimes when you do not know the best route forward, you are best to delay the decision. Not always, but that's the approach I'm taking here.
