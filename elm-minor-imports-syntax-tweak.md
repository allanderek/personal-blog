---
title: "Elm - Minor imports syntax tweak"
tags: elm syntax 
date: 2021-01-20
---

I want to explain a minor tweak to Elm's import syntax, that I cannot find justification for. Before that I'll explain a syntax improvement that I at first thought was a no-brainer, but then came to realise was hard to justify, and certainly not a *no-brainer*. I'll end by saying that despite the fact that I cannot find sufficient justification, I still think my minor syntax tweak should be adopted.

Elm's syntax is famously pretty conservative. Suggesting any new syntax is typically met with pretty fierce resistance. This means to get any new syntax proposal accepted you have to demonstrate that your proposal really can improve some existing code. This is a pretty high bar, but I think that's mostly reasonable. It may seem like a high bar, but really, if you cannot demonstrate many examples of existing code that would be improved by your suggestion then what really is the purpose of your suggestion? Syntax is very [bike-sheddy](https://en.wikipedia.org/wiki/Law_of_triviality), **everyone** has an **opinion** about syntax. However it is very easy to confuse *'I prefer it like this'* with *'this is objectively better'*.

Elm allows you to update one or more of a record value's fields, like so:

```elm
moveRight : Int -> Point -> Point
moveRight amount point =
    { point | x = point.x + amount }
```

The expression before the `|` must have a record type, but syntactically it must be just a bare name, the following are all disallowed:

```elm
{ Point.origin | x = 20 }
{ player.point | x = player.point + 20 }
{ getPoint model | x = 20 }
{ { x = 0, y = 0 } | x = 10 }
{ { p | x = 20 } | y = 10 }
```

The bottom two, are not particularly sensible, if you saw that code in the wild you should suggest changing, they can both be written simpler:


```elm
{ x = 10, y = 0 }
{ p | x = 20, y = 10 }
```

Nonetheless the restriction seems harsh in the first three cases, in all three cases you have to introduce a `let` in order to get around this restriction (and you always can):

```elm
let
    point =
        Point.origin
in
{ point | x = 20 }
```

Similarly for the other two. Still I found this restriction awkward at times, particularly when using a nested record type for the model type, imagine you have the inputs to a login form in a nested record within the main `Model` type, your update handler for input to the login email might be:

```elm
update message model =
    case message of
        LoginEmailInput input ->
        ( { model | loginForm = { model.loginForm | email = input } }
        , Cmd.none
        )
```

To me that seems reasonable, but instead you have to write:
```elm
update message model =
    case message of
        LoginEmailInput input ->
        let
            loginForm =
                model.loginForm
        in
        ( { model | loginForm = { loginForm | email = input } }
        , Cmd.none
        )
```

To me the let-binding here is just noise, but I can see that others' opinions might differ. Anyway I patched the compiler allowing any generic expression before the `|`, but this was rejected. So I set about collecting a catalog of code in the `elm/` and `elm-community/` libraries as well as [Richard Feldman's elm-spa-example](https://github.com/rtfeldman/elm-spa-example). And a surprising thing happened. This catalog got nowhere, there just weren't that many places where my patched compiler would have been much use. Worse still, it would have allowed the two useless expressions above:


```elm
{ { x = 0, y = 0 } | x = 10 }
{ { p | x = 20 } | y = 10 }
```

I *still* think that it would be useful to allow dot-separated names so:

```elm
{ Point.origin | x = 20 }
{ player.point | x = player.point + 20 }
```

As well as **maybe** function application expressions. But I now realise that even this is not a *no-brainer* that I thought the original patch was.

## Finally import statements

In Elm you can alias an imported module to save yourself from typing out the entire name when you use it. So instead of:

```elm
import Json.Decode.Extra

someDefinition =
    ... Json.Decode.Extra.url
```
you can write:
```elm
import Json.Decode.Extra as Decode

someDefinition =
    ... Decode.url
```

However, what you cannot do is give an alias which itself includes a `.` separator:


```elm
import Json.Decode.Extra as Json.Decode
```

This *feels* wrong to me. It feels like you're aliasing a module name with another module name, so you should be able to give it a full module name. You could even make a name longer if you so chose:

```elm
import List as Core.List
```

Now of course this is all a bit silly and very minor, you can of course simply omit the `.` character:

```elm
import Json.Decode.Extra as JsonDecode
import List as CoreList
```

## Status quo bias

I cannot of course provide many or even *any* examples of code that would be improved by this syntax addition. Nevertheless, in contrast to my earlier effort, I still think this deserves consideration and if I were doing the considering I would accept this change. I'm pretty convinced if I were to suggest this, or patch the compiler, such a change would be rejected.

I think it's worth considering the cognitive bias [status-quo bias](https://en.wikipedia.org/wiki/Status_quo_bias).

> Status quo bias is an emotional bias; a preference for the current state of affairs.

In something like a programming language or library, where backwards compatibility is a concern, a preference for the status-quo is probably not a bad thing. However, I think it is good practice to at least consider the possibility that your preference for one option is really only because it is the status-quo. To avoid this, imagine you were making the decision for the first time now. In this particular case, imagine you were designing the `import` statement now, so no such statement yet exists. This is difficult to do (biases are generally difficult to overcome). But it is worth asking yourself the question, if you were designing a new language how, how would you design the import statement? If you honestly would disallow `.`s in the alias, then fine you like the language as is. 

My point here, is that I cannot make a good case for **either** disallowing it, **or** allowing it. I prefer allowing it, though it's hard to say why. But the change shouldn't be immediately rejected unless there is a good reason **not** to make the change. Backwards compatibility is of course a good reason, but that doesn't apply here since all current code would still be legal.


