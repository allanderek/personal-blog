---
title: "Let signatures"
tags: elm programming
date: 2021-05-29
---

A throw-away comment on the elm-radio episode [debugging in elm](https://elm-radio.com/episode/debugging-in-elm) led me to re-evaluate one of my Elm practices. The comment concerned writing a type signature for values/names defined in a `let-in` scope. I've always done this in what seemed to be a *traditional* accepted practice. That is omitting pretty much any and all signatures on names defined within a `let-in` scope. If you look at **most** Elm code you can find on the web, in particular in the elm-lang guide, and in documentation for elm libraries, you will find the same style.

```elm
hellowWorld : String
hellowWorld =
    let
        hello =
            "Hello"
        world =
            "World"

        combineWords =
            String.join " "

    in
    [ hello, world ]
        |> combineWords
```

The throw away comment suggested that you should put type signatures on definitions within let blocks. This had honestly never really occurred to me. However, thinking about it briefly I realised that there is no good reason to have type signatures on top level definitions that doesn't also apply to inner let-defined definitions. 

There is the possibility that you're exporting a top-level definition, but the advice/general practice has never seemed to be to put a signature on all **exported** top level definitions, but put type signatures on all top-level definitions. 

So for the past couple of weeks I've generally been putting type signatures on most let-defined names. I've found several things, mostly that I like doing this and will continue to do so:

1. This actually speeds up finding type errors. Previously the reported error would often been in the `in` expression where a name was used because it's the wrong type. However, the real error is in the actual definition of the name, having the type signature there means I get told about the error where it really is.
2. Particularly for functions this just helps comprehension in the same way that it does for top-level definitions.
3. I'm generally leaving off the signature if the name is a pretty triivally defined name, so the above I might re-write as:


```elm
hellowWorld : String
hellowWorld =
    let
        hello =
            "Hello"
        world =
            "World"

        combineWords : List String -> String
        combineWords =
            String.join " "

    in
    [ hello, world ]
        |> combineWords
```

So I'm still leaving off the type signatures from `hello` and `world`. But it has to be pretty trivial, in partiuclar for *record* types I'm putting the signature, as a common issue is when you add/remove something from the definition of a record type alias, and have to update all the value sites. This is a common situation in which you get the error on the *use* of a defined name, but it's actually the definition that is wrong, and putting the signature on the let-defined name just means that the compiler reports it in the more appropriate place.

So I'm going to continue to do this, thanks again elm-radio.
