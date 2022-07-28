---
title: "More compile time laziness"
tags: elm compilation
date: 2021-01-26
---


Following on [from yesterday](/posts/2021-01-25-maybe-with-default) I think there are various other ways in which a compile-time elm-to-elm optimiser could improve the performance of Javascript code output by the Elm compiler. It goes without saying that this does not necessarily have to be done as a separate tool, it could easily be incorporated into the Elm compiler itself. It is just potentially a lower barrier for entry to do it the separate tool way. It's also a little easier to describe.

I've taken to calling this *compile-time laziness*. You cannot quite get to the same level as a language that is truly lazy (such as Haskell). However, I view this as being something of an analogous phase as a strictness analyser for a lazy language. A strictness analser removes some of the inefficiencies of creating 'thunks' by determining that certain values are co-lazy. That is, `x` and `y` are co-lazy, if evalauating `x` implies that you will evaluate `y`. You can use this to avoid creating a needless thunk for `y`. *Compile-time laziness* in a sense attempts to reach the same *kind* of optimum as a strictness analyser, but instead of starting with **all** values being lazily evalated and determing that some can be made strict, it starts off with all values being strictly evaluated and determines that some can be lazily evaluated. Rather than being lazily evaluted it can instead be **moved**. Here is an example, consider the following code:

```elm
showCouponCode : Coupon -> Discounts -> Html msg
showCouponCode coupon discounts =
    let
        couponIsEmpty =
            String.isEmpty coupon
        couponIsUsed =
            List.any (\discount -> discount.code == coupon) discounts
    in
    case couponIsEmpty of
        True ->
            Html.nothing
        False ->
            case couponIsUsed of
                True ->
                    Html.div
                        [ Attributes.class "used-coupon-code" ]
                        [ Html.text coupon
                        , Icon.tick
                        ]
                False ->
                    Html.div
                        [ Attributes.class "unused-coupon-code" ]
                        [ Html.text coupon
                        , Icon.cross
                        ]

```

The insight here is that whilst `couponIsEmpty` is always evaluated, `couponIsUsed` is ignored if `couponIsEmpty == True`. So we could avoid evaluating the expression `List.any ...`, however, because Elm is a stricly evaluated language this will be evaluated in any case. A compile-time laziness anaylser, which was written as an elm-to-elm transformer, could transform the above code to be:


```elm
showCouponCode : Coupon -> Discounts -> Html msg
showCouponCode coupon discounts =
    let
        couponIsEmpty =
            String.isEmpty coupon
    in
    case couponIsEmpty of
        True ->
            Html.nothing
        False ->
            let
                couponIsUsed =
                    List.any (\discount -> discount.code == coupon) discounts
            in
            case couponIsUsed of
                True ->
                    Html.div
                        [ Attributes.class "used-coupon-code" ]
                        [ Html.text coupon
                        , Icon.tick
                        ]
                False ->
                    Html.div
                        [ Attributes.class "unused-coupon-code" ]
                        [ Html.text coupon
                        , Icon.cross
                        ]

```

In this version of the code, `couponIsUsed` is only evaluated if actually used. It turns out in this case I also think this is actually nicer code. I believe that definitions closer to their use-cases are less likely to be in error, and I also believe it makes sense to limit the scope of declared names to where they are needed. So this highlights another reason it might be useful to write this optimiser as an elm-to-elm transformation. Sometimes the analyser will produce less readable code, and the transformed code should just be handed to the Elm compiler and then thrown away, but sometimes it will produce (in some sense) **better** code, and it can instead be kept.
