---
title: "Laziness"
tags: Elm programming
date: 2021-02-07
---

I have spoken recently about [compile-time laziness](/posts/2021-01-28-safe-dead-code-removal) as well [here](/posts/2021-01-26-more-compile-time-laziness) and [here](/posts/2021-01-25-maybe-with-default). Both Elm and Purescript are pure languages that in theory *could* be compiled as a lazy language. Because there are no side-effects laziness wouldn't change the behaviour of *most* programs and those that it does, it is because of infinite loops that do not need to be calculated. However both have chosen to compile in eager (or strict) fashion, partly to reduce the complexity of the compiler, and partly to allow easier interop with Javascript which is the target of compilation in both cases. 

I think in the examples I linked to above, such as the `Maybe.withDefault <expr>` example, the advantage of general laziness is quite evident, under certain conditions you needlessly compute a value that isn't used. In these cases though it's a pretty simple compile-time transformation that avoids the needless computation. I've heard laziness sceptics wonder why laziness is at all useful before, uttering a phrase something like *"Why would I ever compute a value I do not use anyway?"* (okay so one of those sceptics **may** have been a past me). So I thought I might give a simple example, that while simple enough to convey the meaning of in a blog post, is complicated enough that we can see why we might wish the **compiler** to perform this laziness transformation for us. Whether the compiler does that by a compile-time transformation **or** simply by compiling with lazy semantics. This post should not be taken as an argument that laziness is better than strictness, but purely why one might even consider laziness in the first place.

Suppose you have a list of numbers, and you wish to select the number that has the largest prime as a factor. You do not actually need the largest prime factor, just the number that has the largest prime as a factor. You can do this with something like the following:

```elm
maximumByLargestPrimeFactor : List Int -> Maybe Int
maximumByLargestPrimeFactor numbers =
    let
        largetPrimeFactor n =
            Arithmetic.primeFactor n
                |> List.map abs
                |> List.maximum
                |> Maybe.withDefault 1
    in
    List.maximumBy largestPrimeFactor numbers
```

This uses the [elm-arithmetic library](https://package.elm-lang.org/packages/lynn/elm-arithmetic/latest/), I've defensively used `List.maximum` to find the largest of the prime factors rather than just taking the last item in the returned list since the documentation doesn't assert that they will be in order. I've also fudged the case in which the number has no prime factors (eg. 1). 

Now I think this works pretty well, but it turns out that in the particular application you're writing, the function is called over 99% of the time, with a list in which there is only one element. Of course to find the `maximumBy` of a list with only one element you do not need to compute the `By` value, in this case the prime factors (which might be quite expensive). To avoid this, we could first check if the list is of length one and if so just return that value:

```elm
maximumByLargestPrimeFactor : List Int -> Maybe Int
maximumByLargestPrimeFactor numbers =
    let
        largetPrimeFactor n =
            Arithmetic.primeFactor n
                |> List.map abs
                |> List.maximum
                |> Maybe.withDefault 1
    in
    case number of
        [ one ] ->
            Just one 
        _ ->
            List.maximumBy largestPrimeFactor numbers
```

But hold on, to do the `List.maximumBy` we're making use of the [elm-community/list-extra library](https://package.elm-lang.org/packages/elm-community/list-extra/latest/), perhaps they already perform this optimisation, if so we do not wish to needlessly perform it ourselves. We can check the source code of `List.maximumBy` [here](https://github.com/elm-community/list-extra/blob/8.2.4/src/List/Extra.elm#L268), copied here:

```elm
{-| Find the first maximum element in a list using a comparable transformation
-}
maximumBy : (a -> comparable) -> List a -> Maybe a
maximumBy f ls =
    let
        maxBy x ( y, fy ) =
            let
                fx =
                    f x
            in
            if fx > fy then
                ( x, fx )

            else
                ( y, fy )
    in
    case ls of
        [ l_ ] ->
            Just l_

        l_ :: ls_ ->
            Just <| first <| foldl maxBy ( l_, f l_ ) ls_

        _ ->
            Nothing
```

A ha, so the `list-extra` library is already performing this optimisation for us, note the special case checking for the list of exactly length one. Now in a lazy language, you wouldn't need to explicitly perform this optimisation yourself, and you wouldn't need to check if the library is already performing such an optimisation. A lazy language just naturally does not compute any value that isn't used, and since in a list of one, the `By` value is never compared to any other, it is never used and therefore will never be computed. 

At one stage, before I got into Haskell, I was also of the opinion that laziness was of limited value because I don't tend to needlessly compute values, but once you start noticing these little optimisations you start noticing them everywhere. I found that laziness was pretty liberating because I mostly stopped trying to predict how performant my code would be as it is indeed much more difficult to intuit in a lazy language. I found that this had two benefits:
1. I just wrote the code as I felt was most natural or most clear.
2. I didn't introduce any little 'micro-optimisations' that I **thought** were optimisations. Either I benchmarked the code, or I just didn't introduce such an optimisation.

This second point is important. In an eager language it is common to write code that you **think** is more performant than your first version, only to find that you've deprived the compiler of an opportunity to perform an even better optimisation and as a result your new 'optimised' code is actually slower.
