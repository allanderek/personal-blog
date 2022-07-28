---
title: "Splitting Elm messages"
tags: elm syntax programming
date: 2021-02-03
---

Elm apps, require that we define a single type, usually called `Msg`, to host the type of messages that form a major part of the *'The Elm Architecture'*. This single type is usually a custom/variant type, and because it must host all of the messages that the app may consume, it can get rather large. This leads to a large `update` function. I do not think this necessarily a bad thing, but many beginners to Elm baulk at the idea of large functions, and so they seek solutions to break up their `Msg` type. The basic idea is to make your messages hierarchical, so you have a variant that itself contains a variant. The question is how best to split this up. I'm going to explain why the first instinct in this is usually wrong, suggest a slightly better way, and end up by claiming that the main thing is to remain fluid in your datatypes so that you can best represent whatever the current situation is, rather than cling to an old design for earlier requirements.

The first instinct is usually to group messages together that are somehow related to the same parts of the app.
So you think, let's have all the *login* related messages in a separate type, and then have a single `LoginMsg` variant which holds it.

```elm
type LoginMessage
    = EmailInput String
    | PasswordInput String
    | SendLogin
    | SendLoginResponse (Result Http.Error User)

type Msg
    = UrlChange ...
    | UrlRequest ...
    | LoginMessage LoginMessage
    | ... 
    other app messages
```

This seems like a good idea. The problem comes when you start to think of this as something that might be re-used. Now in fairness, a Login *component* may well satisfy that, but many other *component* seeming parts of your app are really only ever likely to work in that particular app. So let's try to write our `update` functions:


```elm
type alias Model =
    { loginForm : LoginForm
    , user : Maybe User
    , ....
    }
type alias LoginForm =
    { email : String
    , password : String
    , status : LoginStatus
    }

type LoginStatus
    = Ready
    | InFlight
    | Error


update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    case message of
        UrlChange .. ->
            ...
        UrlRequest .. ->
            ...
        LoginMessage loginMessage ->
            updateLogin loginMessage model

updateLogin : LoginMessage -> Model -> ( Model, Cmd Msg)
updateLogin loginMessage model =
    case loginMessage of
        EmailInput input ->
            let
                loginForm =
                    model.loginForm
            in
            ( { model
                | loginForm = { loginForm | email = input }
              }
            , Cmd.none
            )
        PasswordInput .. ->
            .. similar
        SendLogin ->
            let
                loginForm =
                    model.loginForm
            in
            ( { model
                | loginForm = { loginForm | status = InFlight }
              }
            , Requests.sendLogin loginForm
            )
        SendLoginResponse (Ok user) ->
            let
                loginForm =
                    model.loginForm
            in
            ( { model
                | loginForm = { loginForm | status = Error }
              }
            , Cmd.Msg
            )

        SendLoginResponse (Ok user) ->
            ( { model
                    | loginForm = emptyLoginForm
                    , user = Just user
                    }
            , Cmd.none
            )
```

Now this is all a touch, ..., ugly. There is a lot of the pattern of declaring the `loginForm` so that we can use record update syntax to change it. We could of course just define `loginForm` at the top of the function in a `let` declaration before the `case`. Still, the problem I see with this kind of *'breaking up of the update function'* is that it is in name only. You basically have all the same cases that you would have had, it's just that at some point you have the start of a function declaration. However, the **cases** are all **basically** the same as they would have been had they been defined in the main `update` function. So *technically* you have split up the main function, but not really.

The *smoking gun* sign that you're guilty of this kind of faux factorising is the following pattern in your main `update` function:


```elm
        LoginMessage loginMessage ->
            updateLogin loginMessage model
```
Where the whole of your case is just to call the auxiliary function. That means the cases in your auxiliary function must be basically the same as they would have been in your main `update` function.

My main point here is that if you really want to *'break up your update function'*, then focus on the what changes in the model, and which cases require a command. That way you can usefully break up your `update` function. In this case, I see that there are two messages that require no commands and also only update the `loginForm`. So that is how I would choose to break up the main message variant. 


Let's try again:


```elm
type LoginFormMessage
    = EmailInput String
    | PasswordInput String

type Msg
    = UrlChange ...
    | UrlRequest ...
    | LoginMessage LoginMessage
    | SendLogin
    | SendLoginResponse (Result Http.Error User)
    | ... 
    other app messages

type alias Model =
    { loginForm : LoginForm
    , loginStatus : LoginStatus
    , user : Maybe User
    , ....
    }
type alias LoginForm =
    { email : String
    , password : String
    }

type LoginStatus
    = Ready
    | InFlight
    | Error


update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    case message of
        UrlChange .. ->
            ...
        UrlRequest .. ->
            ...
        LoginMessage loginMessage ->
            ( { model
                | loginForm = updateLoginForm loginMessage model.loginForm
              }
            , Cmd.none
            )

        SendLogin ->
            ( { model | loginStatus = InFlight }
            , Requests.sendLogin loginForm
            )
        SendLoginResponse (Ok user) ->
            ( { model | loginStatus = Error }
            , Cmd.none
            )

        SendLoginResponse (Ok user) ->
            ( { model
                    | loginForm = emptyLoginForm
                    , loginStatus = Ready
                    , user = Just user
                    }
            , Cmd.none
            )
updateLoginForm : LoginMessage -> LoginForm -> LoginForm
updateLoginForm message form =
    case message of
        EmailInput input ->
            { form | email = input }
        PasswordInput input ->
            { form | password = input }
```

## Extensible Record Syntax

One thing you could do, to make both forms a bit nicer is to scrap the **loginForm** nested record type. Just have those on the `Model` directly. So we would have something like the following:


```elm
type LoginFormMessage
    = EmailInput String
    | PasswordInput String

type Msg
    = UrlChange ...
    | UrlRequest ...
    | LoginMessage LoginMessage
    | SendLogin
    | SendLoginResponse (Result Http.Error User)
    | ... 
    other app messages

type alias Model =
    { loginEmail : String
    , loginPassword : String
    , loginStatus : LoginStatus
    , user : Maybe User
    , ....
    }
type alias LoginForm =
    { email : String
    , password : String
    }

type LoginStatus
    = Ready
    | InFlight
    | Error


update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    .. as before except ..
        LoginMessage loginMessage ->
            ( updateLoginForm loginMessage model
            , Cmd.none
            )

type alias LoginForm a =
    { a
        | loginEmail : String
        , loginPassword : String
        }

updateLoginForm : LoginMessage -> LoginForm a -> LoginForm  a
updateLoginForm message form =
    case message of
        EmailInput input ->
            { form | email = input }
        PasswordInput input ->
            { form | password = input }
```


There are a couple of reasons against this. The first is relevant here and the second is not relevant for a login form but is a more general point. So firstly, in this updated extensible record syntax, I've glossed over the case in which the login is successful and we have to *'empty'* the login form. In the original version we assumed there was some `emptyLoginForm` defined somewhere, and probably used in the `init` function, so we can use it to reset the nested record type. However, we cannot do that trick with an extensible record, so we're forced to do something like:


```elm
        SendLoginResponse (Ok user) ->
            ( { model
                    | loginEmail = ""
                    , loginPassword = ""
                    , loginStatus = Ready
                    , user = Just user
                    }
            , Cmd.none
            )
```

That's fine, but what if we add something to the login form, suppose you need to take someone's 2FA code as well? Now you have to remember to come back here and reset that. With a nested record type you will be informed of all those places where you need to update/reset that newly added field.

The second reason doesn't apply for login forms, but I have been bitten with. It occurs when you have a single *'object'* let's call it a *'form'* that you use in your application, which then becomes multiple, perhaps a list of such 'objects'. So for example, maybe you have a *comment* form for your online store for people to comment on the store. But then you realise that you want a comment form on each individual product. To drive home the point let's torture our `loginForm` example and suppose that you might have an **admin** login as well. So now your app needs two login forms. With the extensible record form, you're a bit stuck writing the same code twice. But with a nested record type this is quite simple:



```elm
type alias Model =
    { loginForm : LoginForm
    , adminLogin : LoginForm
    , loginStatus : LoginStatus
    , user : Maybe User
    , ....
    }
type Msg
    = UrlChange ...
    | UrlRequest ...
    | LoginMessage LoginMessage
    | AdminLoginMessage LoginMessage
    | ... 
    other app messages

update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    .. as before except ..
        LoginMessage loginMessage ->
            ( { model | loginForm = updateLoginForm loginMessage model.loginForm }
            , Cmd.none
            )
        AdminLoginMessage loginMessage ->
            ( { model | adminLogin = updateLoginForm loginMessage model.adminLogin }
            , Cmd.none
            )
```

So the nice thing here is that I actually got to re-use my auxiliary function, that's generally at least one purpose of factoring out code into its own function, so that you can re-use it.

Admittedly you now have the problem that you have to duplicate the `user` and `loginStatus` fields, and you probably have to duplicate those messages. Of course it would all depend on the exact semantics of your two-login-modes app, which again was a bit of a tortured example. Anyway now you probably want to re-consider your abstractions, which brings me to my final point.

## Be fluid

When splitting up your `update` function, be fluid.  Perhaps we need to consider storing  the `user` and `loginStatus` within the login form, but then use extensible record to write an `updateLoginForm` function that still only operates on the `EmailInput` and `PasswordInput` messages. That keeps our nice factoring out, but allows us to factor out the other functionality of sending the login request and dealing with the response as well.

Be fluid is important here, because one of the biggest risks of *'breaking-up your update function'* is that you begin to make your data types and abstractions more concrete. More concrete means you are less likely to adapt them to the absolute best as new requirements or information come in.


