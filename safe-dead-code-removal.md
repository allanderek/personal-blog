---
title: "Safe dead code removal and compile-time laziness"
tags: elm 
date: 2021-01-28
---

Jeroen Engels of elm-review and elm-radio fame has written [an excellent blog post](https://clouddev.pakk.io:4014/posts/2021-01-26-more-compile-time-laziness) regarding the safe removal of dead code in a purely functional language. The main take-away is that because there are no side-effects, **all** code dependencies are explicit. Because of this it's relatively easy to determine that code does **not** depend on other code, and therefore some code is dead (ie. unused), and can be safely removed.

You can extend this idea, and say the *order* that code is executed in, is only dependent on the explicit dependencies between code. Two days ago I wrote about [compile-time laziness](/posts/2021-01-26-more-compile-time-laziness), I showed that the following code:

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

can be re-written as the following:

    
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

This is a kind of generalisation of Jeroen's point. In this case, `couponIsUsed` cannot be removed entirely because there **is** code that still depends upon it. However it can be **moved** because we know that there are no implicit dependencies on **when** it is evaluated. This is powerful. It allows for some pretty interesting transformations. A commonly acknowledged one is called *'fusion'*. It allows you to avoid some intermediate structures, and/or traverse a structure less often. Consider:

```elm
users
    |> List.map getMainEmail
    |> List.map displayEmail
```

This creates an intermediate list, the result of the first `List.map` call, and also traverses that list. We can avoid both by doing the following:

```elm
users
    |> List.map (getMainEmail >> displayEmail)
```

Note that this changes the execution order. The first version would execute all `getMainEmail` function invocations before then doing all of the `displayEmail` invocations. The second version intersperses them. However, because you're using a purely functional language you **know** that this transformation must be safe, whereby safe we mean results in the same output for any given input.

In the post two days ago I suggested an elm-to-elm transformation tool. One question, given that *elm-review* allows for automated fixes, could we implement such a tool using the excellent code-base of elm-review? We could start by a tool that moves definitions to their inner-most scope, something that often improves the readability/maintainability of the code as well as the performance.


