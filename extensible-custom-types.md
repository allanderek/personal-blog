---
title: "Extensible custom types in Elm"
tags: elm syntax 
date: 2021-01-17
---

There was a [fairly long thread](https://discourse.elm-lang.org/t/idea-extensible-union-types-and-benefits-they-bring-for-real-world-elm-code/6118) on the Elm discource regarding a proposal to add extensible custom (union) types to Elm. I thought I would try to summarise a little, the current status. In brief, I think currently the cons outweight the pros, but most of the cons are *uncertainty* rather than *definite disadvantages*  so it is possible that some further work could tip the balance. I'll describe here very briefly what an extenible custom type is, and then the pros and cons, why I think the cons outweight the pros at present, and finally a path towards overturning that.


## Extensible record types

In Elm you can write a record type such as:

```elm
type alias Point = { x : Int, y : Int }
```

But you can also write an *extensible* record type such as:

```elm
type alias Point a = { a | x : Int, y : Int }
```

The first means that something of type `Point` has exactly two fields `x: Int` and `y : Int`, whilst the second states that a value must have *at least* those two fields, but may have more. You can later instantiate the missing fields:

```elm
type alias Point3D = Point { z : Int }
```

Although you can use extensible record types in this way to model data, the standard advice is to use extensible record types to constrain function inputs. In addition extensible record types are used during type inference, for example a function (lacking a type signature) can be typed as accepting an extensible record type:

```elm
showLink link =
    Html.a
        [ Attibutes.href link.url ]
        [ Html.text link.content ]
```

Can be typed as: `{ a | url : String, content : String } -> Html msg` which makes things a little easier when unifying the applications of the `showLink` function. 

## What are extensible custom types?

Extensible custom types would allow you to do an analagous thing with custom types. However, note that the custom types currently introduce a new type, as opposed to record types which introduce a type *alias*. So exactly how this would work is debatable, however I'm going to go with what I think is the most natural, which is to make custom types similar to record types, so therefore an alias, and extensible:

```elm
type alias Responses 
    = Elm
    | Typescript
    | Purescript
```

Would be a normal custom type, but an extensible one would be written as:


```elm
type alias Responses a
    = a
    | Elm
    | Typescript
    | Purescript
```

Now similarly a function can be typed as **returning** an extensible custom type:

```elm
parseResponse input =
    case input of
        "Elm" -> Just Elm
        "Typescript" -> Just Typescript
        "Purescript" -> Just Purescript
        _ -> Nothing

```

Can be typed as `String -> Maybe (a | Elm | Typescript | Purescript)`.
Quick thing to notice here; record type A is a subtype of record type B if A has all the fields that B has and possibly more. But custom type A is a subtype of custom type B if **B** has all the variants of A and possibly some more. To see why, it's okay for a function which accepts a record type B to accept record type A if A has **at least** all the fields that B has. But for custom types it's okay for a function which accepts custom type B, to accept a value of type A if A has **at most** the same variants as B. In particular consider the function:

```elm
showConstrainedReponse response =
    case response of
        Elm -> "Elm"
        Typescript -> "Typescript"
        Purescript -> "Purescript
```

Cannot have type `a | Elm | Typescript | Purescript -> String` because if you pass a variant not in `Elm | Typescript | Purescript` the program would crash. However if I write the function as:


```elm
showLiberalReponse response =
    case response of
        Elm -> "Elm"
        Typescript -> "Typescript"
        Purescript -> "Purescript
        Other s -> s
```

I can, naturally, pass this function a value of type `Elm | Typescript | Purescript | Other String` but I could also pass it a value of type `Elm | Typescript | Purescript` since that is a subtype of the former. So in particular I could do the following:

```elm
input
    |> parseResponse
    |> Maybe.map showLiberalResponse
```

Because the type of `showLiberalResponse` is `Elm | Typescript | Purescript | Other String -> String`.

## Use cases

I find natural use-cases are a little hard to come by. In Elm, people often end up with a single large `Msg` message custom variant type. This leads to long `update` functions. If you want to break this up, one thing you can do is:

```elm
type Msg
    = MessageClass1 MessageClass1
    | MessageClass2 MessageClass2
    etc.
```

So you might do something like:

```elm
type Msg
    = SubscriptionMsg SubscriptionMsg
    | LoginMsg LoginMsg
    | ProfileMsg ProfileMsg
    etc.
```

Then you can write your main update function as:

```elm
update message model = 
    case message of
        SubscriptionMsg subMessage ->
            updateSubscriptionMessage subMessage model
        LoginMsg loginMessage ->
            updateLoginMessage loginMessage model
        ....
```

But because the `Cmd` returned has to be of type `Msg` you have to use `Cmd.map` on the result of each sub-function, **or** write each sub function as:

```elm
updateSubscriptionMessage : SubscriptionMsg -> Model -> ( Model, Cmd Msg)
```

It tends to end up being the latter, because the sub-functions can return commands that produce messages outside of the domain of the sub-function.

This scheme would be a bit more elegant, with extensible custom variant types, but because this **can** be done without extensible custom variant types, it makes the case for them a little weaker.


## Pros

1. Clearly there are situations that call for extensible custom types.
2. I like the elegance of the symmetry between extensible record types and extensible custom types. 
3. There are probably good cases of being able to extend custom variant types that are defined in some third-party library that you cannot control.

## Cons

1. Unlike extensible record types this would increase the complexity of type inference. Albeit **because** of the increased expressivity. Currently you cannot write a function that matches over a sub-set of a full custom type's variants. There are two large uncertainties here, one is how extensible custom types would affect the complexity (speed, space requirements) of the type inference argument and the second is how it might affect the quality of compiler error messages.
2. Although you can think of use-cases, it's a little more difficult to think of use-cases that cannot be solved by structuring your types differently. Again that is **possible** but difficult. 
3. Any addition to the language comes with the possibility of introducing strange interplays that you didn't think about. 


## Conclusion

So I think currently the cons outweight the pros. It's relatively easy to come up with simple examples where your code is a little nicer, but much more difficult to come up with examples where you have tangible benefits. Given Elm's focus on safety, and the common mantra to *"Make impossible states impossible"* I think some clear examples would show some code A that guarantees some property X that cannot be guaranteed given the current type system. That seems like a high bar, but if you cannot come up with such an example (or indeed several) then you're basically arguing that it should be included in the language because it's "nicer". 

However, notice that the two major cons are mostly uncertainty. It only requires work to disprove either of them (if indeed they are false). If you want to see extensible custom types in Elm you have a route to this, it's just not terribly attractive:
1. Fork the compiler and implement custom union types
2. Benchmark the unforked and forked compilers' type-inference algorithm on at least the current tests within the compiler A lot of work was done in 0.19 to increase the compiler speed, and much of that was related to type inference, so I suspect there are a bunch of good test cases there.
3. Use your forked compiler to author some compelling example use cases.


