---
title: "My opinionated import style"
tags: elm syntax 
date: 2021-01-10
---

In Elm it's possible to import a name from a module in two ways, you can either import the module and expose the name, or import the module, perhaps aliasing the module name, and then qualify the use of the target entity.

This seems common in documentation, I'm not sure why it seems to be that having unqualified names "looks better", and there is certainly a sense in particular for the `elm/html` library that the resulting code more resembles the HTML that is essentially being described.

```elm
import Html.Attributes exposing (attribute, value, type_, attribute)
import Html exposing ( Html, input, text )
import Html.Events exposing (onInput)

showEmailInput : String -> Message -> Html Message
showEmailInput valueString message =
    input
        [ value valueString
        , type_ "email"
        , attribute "role" "user-email"
        , onInput message
        ]
        []
```

I personally dislike this style. I prefer to essentially qualify every name, aside from types that share the same name as the module (and occasionally a few select common types). So I would re-write the above as:


```elm
import Html.Attributes as Attributes
import Html exposing ( Html, Attribute)
import Html.Events as Events

showEmailInput : String -> Message -> Html Message
showEmailInput value message =
    Html.input
        [ Attributes.value value
        , Attributes.type_ "email"
        , Attributes.attribute "role" "user-email"
        , Events.onInput message
        ]
        []
```

I realise that to some tastes this has a lot of *'noise'*, but I think it has these advantages, the first two of which are minor:
1. There is less moving between the code and the top of the module to change the imports (to include more used names in the exposing list), and it's also more likely that your import list becomes stale, in that you import names that are never used.
2. When reading code I can immediately tell whether a value is imported or defined within the current module
3.  There is a tension between assuming that your module's exported names will be imported qualified or not. You will occasionally be forced into qualifying a name because it clashes with a name imported from another module. Because of this, if you just always qualify your imports you do not need to change your style.


I find that if you do *not* do this, you design your module exports as if they will be imported unqualified.  This results in suffixing/prefixing a lot of your exported names. So that you end up with something like:

```elm
module Components.Email exposing
   ( viewEmailInput
   , viewEmail
   , validateEmail
   , defaultEmail
   )
```


In these situations I feel like what the person is doing is re-inventing a qualified name but in a way that isn't checked by the compiler or likely to be done very consistently. Notice here that the 'Email' qualifier is sometimes neither a prefix nor a suffix as in `viewEmailInput`. Now if, when you come to use this module, you decide you would rather not spend time maintaining the list of imports and use qualified imports you end up using names such as `Email.viewEmailInput` and `Email.validateEmail`. If on the other hand you just assume that your module will always be imported qualified you end with use sites that look like `Email.viewInput` and `Email.validate`, which to my mind are a lot nicer than **either** `viewEmailInput`, `validateEmail` **or** `Email.viewEmailInput`, `Email.validateEmail`.

The exception I make for this is when you import a type that is named the same as the module, so if the `Email` module exposes an `Email` type, I just import that exposed, that avoids the use site of `Email.Email` which seems redundant. You'll note in the example given above I've also exposed the type `Attribute`. That's a rare exception for just a really common name, I don't have to worry about this as I know where the `Attribute` type is defined.

So that's my highly opinionated scheme for imports. It's a shame that the language does not enforce such a scheme, even if it was a scheme that was highly unlike my preferred one. I can get used to pretty much any convention, but importing two modules from separate libraries that assume different schemes can be less than optimal. I'm not saying that Elm *should* enforce an import scheme since that might be quite difficult to do, I'm saying it's a shame that it isn't easier and more natural to enforce such a scheme.

Finally [here is a relevant post on the Haskell-cafe mailing list](https://mail.haskell.org/pipermail/haskell-cafe/2008-June/043986.html).
