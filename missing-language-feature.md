---
title: "Missing language feature - Remove from scope"
tags: elm programming
date: 2021-02-17
---

Today I'm going to talk about a language feature that I think is missing from **most** languages. I know of at least one in which it exists, I do not know of any *functional* langauges in which it exists. The language feature I'm talking about is the removal of a name from the scope. Let me first talk about a situation that is ripe for bugs, and then introduce the idea of removing a name from the scope so that you do not use it mistakenly.

A common problem is using randomness in a functional language, I've [written about this before](/posts/2021-01-23-immutabilit-bugs) as being a source of bugs. The reason is, that using a pseudo random number generator with a seed, is actually a really nice problem for having some self-contained state, ie. an object. Without that, you have to remember to store the updated seed back on your model, if you fail to do that, then you will at some point use the same iteration of the pseudo-random function. So you might have something like the following in your `update` function in an Elm application:

```elm
rollDice : Model -> Model
rollDice =
    let
        generator =
            Random.int 1 6
        (newDice, newSeed )=
            Random.step generator model.seed
    in
    { model
        | dice = newDice
        , seed = newSeed
    }

update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        ( rollDice model
        , Cmd.none
        )
    ...
```

All great so far. Now suppose if the dice roll is a six you also want to double the player's score.


```elm
update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        let
            newModel =
                rollDice model
        in
        ( case newModel.dice == 6 of
            True ->
                { newModel | score = newModel.score * 2 }
            False ->
                newModel
        , Cmd.none
        )
    ...
```

Now notice, that the term `newModel` is used four times, we can if you prefer refactor so that we always update the score, I think of this as *'restricting the scope of the conditional'*


```elm
update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        let
            newModel =
                rollDice model
            newScore =
                case newModel.dice == 6 of
                    True ->
                        newModel.score * 2
                    False ->
                        newModel.score
        in
        ( { newModel | score = newScore }
        , Cmd.none
        )
    ...
```
This is *arguably* better, but the `newModel` name is still used four times. Any one of those could be mistakenly written as `model`. I find that after refactoring this is much more likely. For example, suppose on every roll of the dice your score is increased by one regardless, then only later is the new rule introduced which doubles the score if you roll a six. Note that in both instances of `newModel.score` you could replace that with `model.score` and you wouldn't fail any tests **now**. But if the requirements change again, this could well be a bug-in-waiting.


My whole point in this, is that directly after the binding of `newModel` we would quite like to remove `model` from use, that is, remove it from the scope. Any use after that point is probably a bug. It's possible that you do want to refer to the *old* model, but the language gives us no way to express that that is likely a bug.


In python, removing a name from the scope is actually possible using the `del` keyword:

```python
>>> x = [1,2,3]
>>> y = x
>>> del x
Traceback ...
NameError: name 'x' is not defined
>>> y
[1, 2, 3]
```

A major problem with this idea in many functional languages, Elm included, is that the order of the definitions is not semantic. In particular the compiler is free to move them around if it sees fit. This means that it is somewhat dubious to talk about removing a name from the current scope since you would need to remove it from the entire scope, so you wouldn't be able to use it to create the new name. However, a way around this would be to have the entire rest of the expression in a new `let` block, you could imagine some definition like this:


```elm
update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        let
            newModel =
                rollDice model
            scoredModel =
                let without model
                    newScore =
                        case newModel.dice == 6 of
                            True ->
                                newModel.score * 2
                            False ->
                                newModel.score
                in
                { newModel | score = newScore }
        in
        ( scoredModel
        , Cmd.none
        )
    ...
```

Whereby a use of `model` within the inner `let` that uses an imaginary syntax to remove `model` from the scope `let without model` would be an error. If Elm allowed you to re-define a name you could somewhat fake this by doing:


```elm
update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        let
            newModel =
                rollDice model
            scoredModel =
                let 
                    model = ()
                    newScore =
                        case newModel.dice == 6 of
                            True ->
                                newModel.score * 2
                            False ->
                                newModel.score
                in
                { newModel | score = newScore }
        in
        ( scoredModel
        , Cmd.none
        )
    ...
```

In that case, any use of `model` without the inner scoped `let` would likely be a type-error since `model` has been re-declared to be the unit-value. This would certainly prevent you from using it in any of the positions `newModel` is used here.

Still though, both with the imaginary `let without` syntax, and with the (disallowed) rebinding, this all feels rather clunky. I cannot come up with a nice syntax for expressing the fact that a name should not be used further in a let binding. Although functional let definitions do tend to be unordered, they are still read in by the compiler in the order they are written so in theory we could still have a construct to hide a name in the remainder of the bindings in a `let`. So I could imagine writing something like the following, whereby `hide <name>` syntax means hide the given name from the remaining declarations **and** the *in-expression*:


```elm
update : Msg -> Model -> ( Model, Cmd Msg)
update message model =
    ...
    RollDice  ->
        let
            newModel =
                rollDice model

            hide model

            newScore =
                case newModel.dice == 6 of
                    True ->
                        newModel.score * 2
                    False ->
                        newModel.score
        in
        ( { newModel | score = newScore }
        , Cmd.none
        )
    ...
```


Ultimately I think this feature is just too awkward to implement in any elegant language. It wouldn't be used very often, probably not as often as it should be. So likely the bother of complicating the language is just not worth the extra expressivity. Unless of course someone else comes up with a really elegant syntax.
