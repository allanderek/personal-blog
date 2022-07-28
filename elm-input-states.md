---
title: "Elm input states"
tags: elm programming
date: 2021-03-30
---

A small design decision has come up whilst developing with Elm on both the front-end and the back-end. The issue concerns an input that has some UI state, that doesn't need to be transfered from front-end to the back-end (or vice versa). So I'm going first explain an example of an input that might have some associated UI state that you need to keep track of. Then I'm going to explain how you might represent this has part of a larger entity that needs to be encoded into JSON, and decoded from JSON to send to and from the front-end and back-end.


## Integer inputs

In Elm if you have some input that you want to store as an integer, you have a small problem, because the user might make some intermediate stage that isn't a number, in particular the empty string. So typically what you do is store both the raw input string, and the parsed integer, as a `Maybe Int` since of course the input might not parse.

```elm
type alias IntInput =
    { input : String
    , value : Maybe Int
    }
```

You do the obvious thing upon new input:

```elm
updateIntInput : String -> IntInput
updateIntInput input =
    { input = input
    , value = String.toInt input
    }
```



## An integer input container

Now suppose you have some data structure that you wish to pass back and forth between the front-end and back-end.

```elm
type alias Driver =
    { name : String
    , number : Int
    }
```

You can easily define an encoder and decoder:

```elm
encodeDriver : Driver -> Encode.Value
encodeDriver driver =
    [ ( "name": driver.name |> Encode.string )
    , ( "number": driver.number |> Encode.int )
    ]
        |> Encode.object 

driverDecoder : Decoder Driver
driverDecoder =
    Decode.succeed Driver
        |> Decode.andMap (Decode.field "name" Decode.string)
        |> Decode.andMap (Decode.field "number" Decode.int)

```
Now the problem is. Suppose your app allows you to *create* drivers and send them to the back-end. The existing drivers are also sent from the back-end to the front-end. The problem is, how do you represent the list of drivers in  your front-end, bearing in mind that you also have to represent their associated `IntInput`s.

## Storing the UI state separately

You can store the int inputs, separately to the drivers.

```elm
type alias Model =
    { ...
    , drivers : List Driver
    , driverNumberInputs : List IntInput
    ...
    }
```

In this case whenever you update the number in a driver edit form, you have to update the input, which might also cause you to update the actual driver:

```elm
update message model =
    case message of
        ....
        DriverNumberInput index string ->
            let
                newDriverInput =
                    updateIntInput input
                newDriverInputs =
                    List.setAt index newDriverInput model.driverNumberInputs
                newDrivers =
                    case newDriverInput.number of
                        Nothing ->
                            model.drivers
                        Just number ->
                            List.updateAt index (\d -> { d | number = number } ) model.drivers
            in
            ( { model 
                | drivers = newDrivers
                , driverNumberInputs = newDriverInputs
                }
            , Cmd.none
            )
        ... 
```

In a real application, I would probably represent the driver inputs using a dictionary, since all the drivers perhaps have an `id` field. Maybe you could just use the name for that (what happens if two drivers really have the same name?). Anyway, you can then draw the edit field using the driver input, if no driver input is in the dictionary for that driver you can take that to mean that it has not yet been edited.

```elm
    let
        driver =
            ...
        driverNumberInput =
            Dict.get driver.name model.driverNumberInputs
                |> Maybe.withDefault 
                    { input = String.fromInt driver.number
                    , number = Just driver.number
                    }
    in
    Html.input
        [ Attributes.value = driverNumberInput.input
        ...
        ]
        []
```


## Using a type-argument

The risk with the above solution is that the driver representation, without due care, could become out-of-sync with the driver number inputs. Another possibility is to generalise the representation of a driver:


```elm
type alias Driver a =
    { name : String
    , number : a
    }

encodeDriver : (a -> Int) -> Driver a -> Encode.Value
encodeDriver getNumber driver =
    [ ( "name": driver.name |> Encode.string )
    , ( "number": driver.number |> getNumber |> Encode.int )
    ]
        |> Encode.object 

driverDecoder : (Int -> a) -> Decoder (Driver a)
driverDecoder unparseNumber =
    Decode.succeed Driver
        |> Decode.andMap (Decode.field "name" Decode.string)
        |> Decode.andMap (Decode.field "number" (Decode.int |> Decode.map unparseNumber))

```

With this approach the front-end can see drivers as containing `DriverNumberInput`s, whilst the back-end, can ignore that and just use an `Int` as the type parameter for the driver number, so the front-end defines:

```elm
type alias FrontEndDriver = Driver DriverNumberInput

frontEndEncodeDriver : FrontEndDriver -> Encode.Value
frontEndEncodeDriver =
    encodeDriver .number

frontEndDriverDecoder : Decoder FrontEndDriver
frontEndDriverDecoder =
    driverDecoder updateIntInput
```

And the back-end defines:


```elm
type alias BackEndDriver = Driver Int

backEndEncodeDriver : BackEndDriver -> Encode.Value
backEndEncodeDriver =
    encodeDriver identity

backEndDriverDecoder : Decoder BackEndDriver
backEndDriverDecoder =
    driverDecoder identity
```

The downside to this approach is that it can get a bit complicated if your data type has different kinds of values that all require different UI states. In particular sometimes you basically want some UI for **all** inputs, which records whether or not the input has been 'blurred' as this is then used to decide whether or not to display an error message for invalid input.


## Conclusion

At this point, I don't know which of the two are better, but my feeling is that the latter approach is safer.
