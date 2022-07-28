---
title: "Immutability bugs"
tags: functional programming immutability bugs
date: 2021-01-23
---

I am a paid up member of the immutability appreciation society. I believe programming in an immutable language reduces bugs, as well as potentially helps the compiler optimise your code (though perhaps at the cost of not being able to do similar optimisations yourself). I also rather suspect that immutability has a role to play in concurrent code, but that's somewhat irrelevant for today.

However, although I feel that immutability *reduces* bugs, I do not think of it as a strict subset. A strict subset would imply both that, some bugs introduced using a mutable language are simply not possible, or at least less likely when using an immutable language **and also** there are no bugs introduced using an immutable language that would be impossible or less likely when using a mutable language. It's the second part I disagree with. I think most *immutable programmers* know this, but it's worth reminding ourselves of this. It's very easy to get comfortable in our land of the fewer bugs.

My favourite example of this kind of bug is a random number generator. In pure Javascript if you want to get a random number then you can simply call `Math.random()`. In Elm it's a little more complicated, you can either use commands (which can be a inconvenient within some algorithm), or you can manually step a generator. However, if you do the latter, you have to do some manual state management.

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
```

So the bug that would be easy to introduce here would be to forget to update the model with the new seed:


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
    }
```

You probably catch this bug pretty quickly unless you're using the seed on the model for other things as whatever you're rolling the dice for would become pretty predictable, but you can have a more subtle bug such as:


```elm
rollTwoDice : Model -> Model
rollTowDice=
    let
        generator =
            Random.int 1 6
        (newLeftDice, firstNewSeed ) =
            Random.step generator model.seed
        (newRightDice, secondNewSeed ) = 
            Random.step generator model.seed
    in
    { model
        | dice = newDice
        , newSeed = secondNewSeed
    }
```
In this case we should have used `firstNewSeed` in the second call to `Random.step`, a similar bug would be:

```elm
rollTwoDice : Model -> Model
rollTowDice=
    let
        generator =
            Random.int 1 6
        (newLeftDice, firstNewSeed ) =
            Random.step generator model.seed
        (newRightDice, secondNewSeed ) = 
            Random.step generator firstNewSeed
    in
    { model
        | dice = newDice
        , newSeed = firstNewSeed
    }
```
This case has a correct second call to `Random.step` but the returned model is updated with the first returned seed rather than the second one.

In all three of these cases, some static analysis would help you find this bug, as it would report unused but declared variables. In the three cases `newSeed`, `firstNewSeed` and `secondNewSeed` respectively would be reported as an unused variable. Still, more complicated cases could fool the static analysis.

The point here, is that none of these bugs would be anywhere near as likely in a mutable language, static analysis or not. Again, I'm convinced that immutability reduces bugs, it just doesn't do so as a strict subset.

