---
title: "Elm extensible record syntax and warnings"
tags: elm syntax maintenance
date: 2021-01-05
---

Elm has extensible records, you can write a type like `{ a | x : Int }` which means a record type that has **at least** a field named `x` of type `Int` but may have other fields as well. The general advice seems to be not to use these for data modelling but instead use them to narrow the type of function arguments. See [this Richard Feldman talk](https://www.youtube.com/watch?v=DoA4Txr4GUs&feature=youtu.be) and, for example, [this Charlie Koster blog post](https://ckoster22.medium.com/advanced-types-in-elm-extensible-records-67e9d804030d).


I use this all the time. In particular a lot of my view functions take, as the first parameter, a value named `model` and this is typically typed as an extensible record that is satisfied by the application's `Model` type (ie. the `Model` type is a subtype of this extensible record, usually meaning that it has a subset of the fields of the `Model` type, obviously with compatible types). One nice thing about this is that I can change the requirements of my view functions without changing all the call sites, since the call sites typically just pass in the entire model. 

# Unnecessary local constraints

There is a mild wrinkle in this approach though. The compiler itself will not warn about any unnecessary constraints on your function type, the way it would warn about an unused parameter. For example, suppose you wanted to show a logged-in user in your application, you might write a function like:

```elm
viewLoggedInUser : UserName -> Email -> Html msg
viewLoggedInUser userName email =
    div
        [ class "logged-in-user" ]
        [ text username
        , text " ("
        , text email
        , text ")"
        ]
```

Now, you could re-write this as:
```elm
viewLoggedInUser : { a | loggedInUserName : UserName, loggedInEmail : Email } -> Html msg
viewLoggedInUser model =
    div
        [ class "logged-in-user" ]
        [ text model.loggedInUserName
        , text model.loggedInEmail
        , text " ("
        , text ")"
        ]
```

All good, either is fine. Now suppose you decide that you don't need to show the email address, the user knows their own email address probably. In the first first version you'll modify the code to:

```elm
viewLoggedInUser : UserName -> Email -> Html msg
viewLoggedInUser userName email =
    div
        [ class "logged-in-user" ]
        [ text username
        ]
```
This will give you a warning about an unused parameter `email`. Now if you fix this, you'll also have to fix any call sites. The second version has the nice property that you can remove the requirement that the (single) argument has a field `loggedInEmail` and you do not need to change any of the call sites: 

```elm
viewLoggedInUser : { a | loggedInUserName : UserName, loggedInEmail : Email } -> Html msg
viewLoggedInUser model =
    div
        [ class "logged-in-user" ]
        [ text loggedInUserName
        ]
```

However, the mild wrinkle is that, in contrast to separate function arguments, the compiler itself will not warn you about unneeded constraints. You can fix this a bit by using unpacking arguments syntax:
```elm
viewLoggedInUser : { a | loggedInUserName : UserName, loggedInEmail : Email } -> Html msg
viewLoggedInUser { loggedInUserName, loggedInEmail } =
    div
        [ class "logged-in-user" ]
        [ text loggedInUserName
        , text loggedInEmail
        , text " ("
        , text ")"
        ]
```

Now if you remove the use of `loggedInEmail` you'll again get a warning, it won't specifically warn you to update the type signature, so you can still make the mistake of just removing the unnecessary *name*, changing it to the following definition which will give you no warning:

```elm
viewLoggedInUser : { a | loggedInUserName : UserName, loggedInEmail : Email } -> Html msg
viewLoggedInUser { loggedInUserName } =
    div
        [ class "logged-in-user" ]
        [ text loggedInUserName
        ]
```

## Unnecessary derived (non-local) constraints

Worse than this, sometimes your contraint didn't involve any name used in the current function, but in one that you call. In this example, one call site might be:

```elm
viewUserBar : { a | loggedInUserName : UserName, loggedInEmail : Email, loggedInProfileImage : Image } -> Html msg
viewUserBar model =
    div
        [ class "user-bar" ]
        [ viewLoggedInUser model
        , viewImage model.loggedInProfileImage 
        , logoutButton
        ]
```
Now the constraint on the model in the `viewUserBar` requires a `loggedInEmail` field only because there is such a constraint on `viewLoggedInUser` which is called. If you update `viewLoggedInUser` so that that it is no longer a requirement nothing helps you to clean up the constraint here.

Once again, you can mitigate this, this time by using a type alias something like:

```elm
type alias LoggedInUserModel a =
    { a 
        | loggedInUserName : UserName
        , loggedInEmail : Email
        }
viewLoggedInUser : LoggedInUserModel a -> Html msg
...

viewUserBar : LoggedInUserModel { a | loggedInProfileImage : Image } -> Html msg
...
```

Still, I find that all of this can mean that your function constraints become a bit unmaintained, to the point that eventually you can end up with what seems to be a cycle of dependencies but that cycle can be broken easily because one part of the cycle is not required. 

I do not have a solution for this, it would be great if the compiler warned about unnecessary constraints. Otherwse I guess it would be possible to write an [elm-review](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/) rule for this, but it would be hard, because you would essentially have to write a type inference engine.

## Conclusion

I nevertheless recommend using extensible record syntax to narrow the type of your function arguments. Try to be conscientious and use type aliases where possible, but it does mean coming up with a bunch of extra names. Additionally I guess using record pattern matching syntax can help keep the constraints in check a bit. (I personally prefer not to use record pattern matching syntax but I'm not sure why, I have no rational reason for that I just prefer using dot to access the fields). 
