---
title: "SimulatedHttp, functors and sed"
tags: elm programming
date: 2022-03-11
---

In this post I'm going to describe an awkwardness encountered when using [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) to simulate HTTP events in the program being tested. I will then describe ML functors, a feature of the SML/Ocaml module system and show how these would solve the awkwardness. I'll then show how it's pretty simple to hack together a "poor-person's-functor" and use that to solve the aforementioned awkwardness.

### An awkwardness when simulating HTTP for testing

If you haven't used [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) to test an entire Elm program I recommend trying it out. I've found that it not only does the obvious part of helping to build robust tests for Elm programs, but also helps me structure the Elm program in a way that is better for testing, but also just generally better.

In order to use [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) you have to write an auxiliary `update` function for your program that returns a `ProgramTest.SimulatedEffect` rather than a `Cmd`. In order to do this without completely re-writing the program (and thus more or less negating the point of the testing), [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) advises you to re-write your `update` function so that it returns a custom `Effect` type. Then in the real program you translate that `Effect` into a `Cmd Msg`, and for the tests you translate it into a `ProgramTest.SimulatedEffect`. This is best described with some Elm types:

```elm
type Effect
    = GetPosts
    | GetPostComments
update : Msg -> Model -> (Model, Effect)

perform : Model -> Effect -> Cmd Msg

simulate : Model -> Effect -> SimulatedEffect Msg
```

Now, helpfully, the modules in [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) for creating simulated effects, tend to have the same API as their equivalent modules for real commands (in some cases they are incomplete as yet). For example `SimulatedEffect.Http` has the same API as `Http` from `elm/http`. This means it's relatively **trivial** to write the `perform` and `simulate` modules. In fact, they **can** be, essentially identical:


```elm
perform : Model -> Effect -> Cmd Msg
perform model effect =
    case effect of
        GetPosts ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostsReceived (Decode.list Post.decoder)
                }
        GetPostComments ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostCommentsReceived (Decode.list Post.commentDecoder)
                }

simulate : Model -> Effect -> SimulatedEffect Msg
simulate model effect =
    case effect of
        GetPosts ->
            SimulatedHttp.get
                { url = "/api/posts"
                , expect = SimulatedHttp.expectJson PostsReceived (Decode.list Post.decoder)
                }
        GetPostComments ->
            SimulatedHttp.get
                { url = "/api/posts"
                , expect = SimulatedHttp.expectJson PostCommentsReceived (Decode.list Post.commentDecoder)
                }
``` 

In this very simple example the two functions are essentially identical aside from using the `Http` and `SimulatedEffect.Http` modules. This is generally true, even although the logic might be much more complicated. For example, you may have to check if the user is logged in, and if so send an authenticated request. Additionally, **some** effects, might not be simulated at all, for example Dom effects such as focusing are not yet simulated (though you could fake with a simulated port call). Anyway the point is, there ends up being quite a bit of duplicated code.

### ML functors

ML has a pretty powerful module system. In truth, although it is powerful, even when using O'caml to develop a compiler, I still very rarely found that I needed to reach for the full power of the module system. Functors just didn't come up very often. What are functors? They are essentially the equivalent to a module, that a function is to a value. So you can think of them as functions over modules. So you can write a module `A`, which takes another module `B` as an argument. The argument is specified as a module signature. This means that `A` is now a functor. You can apply the functor `A` to more than one other module as long as you have multiple modules that satisfy the signature `B`. The normal use cases for functors are pretty similar to the use cases for Haskell's type classes.

A common example is a dictionary/set, here is such an example written in a fantasy version of Elm with functors (and multiple modules within a single file):

```elm
signature Compare
    type Item
    compare : Item -> Item -> Order

functor Set (Item : Compare)
    type Set
        = Empty
        | Node Set Item Set
    empty : Set
    empty =
        Empty
    add : Item.Item -> Set -> Set
    add item currentSet =
        case currentSet of
            Empty ->
                Node Empty item Empty
            Node left nodeItem right ->
                case Item.compare item nodeItem of
                    LT ->
                        Node (add item left) nodeItem right
                    GT ->
                        Node left nodeItem (add item right)
                    EQ ->
                        currentSet
    ...

module IntCompare
    type alias Item = Int
    compare : Item -> Item -> Order
    compare = Core.compare
module IntSet = Set(IntCompare)
```

Obviously a real implementation would have the other common `Set` functions, and you could use this to make `Set`s of things that aren't in Elm's `comparable` type class, by actually writing your own `compare` function. 
You can read the documentation for [O'caml's module system here](https://ocaml.org/manual/moduleexamples.html)

### Effects, SimulatedEffects, and Functors

Hopefully it is pretty obvious how this solves our awkwardness with requests and simulated requests. Because the SimulatedHttp module from [elm-program-test](https://package.elm-lang.org/packages/avh4/elm-program-test/latest/) has the same API (or signature) as the standard Http module, you can easily write a **functor** that, given either of those two modules, produces a `Perform` module that has the correct type. Again using our fantasy version of Elm with functors:

```elm
type Effect
    = GetPosts
    | GetPostComments
update : Msg -> Model -> (Model, Effect)

module InterpretEffects(Http : <suitable-signature>, Result : <signature with return type>)
    perform : Model -> Effect -> Return.Cmd Msg
    perform model effect =
        case effect of
            GetPosts ->
                Http.get
                    { url = "/api/posts"
                    , expect = Http.expectJson PostsReceived (Decode.list Post.decoder)
                    }
            GetPostComments ->
                Http.get
                    { url = "/api/posts"
                    , expect = Http.expectJson PostCommentsReceived (Decode.list Post.commentDecoder)
                    }
module Perform = InterpretEffects(Http, ( Cmd ) )
module Simulate = InterpretEffects(SimulatedEffect.Http, ( SimulatedEffect ))
```

I've had to fudge this a bit because the return types of both modules (`Cmd` and `SimulatedEffect`) are not defined in the respective `Http` module, but you get the idea. 

### Elm doesn't have functors

However, it's pretty simple to fake them with the use of the unix program `sed`. Just write the `Perform` module as you would, and then copy into a Simulate module, whilst modifying only the parts that change. The use of an `import as` can make this especially doable. First the `Perform` module, remember, this is translating our `Effect` custom type into the Elm's standard library `Cmd` type:

```elm
module Perform exposing (perform)

import Model exposing (Model)
import Model exposing (Msg)
import Http

perform : Model -> Effect -> Cmd Msg
perform model effect =
    case effect of
        GetPosts ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostsReceived (Decode.list Post.decoder)
                }
        GetPostComments ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostCommentsReceived (Decode.list Post.commentDecoder)
                }
```
Now the `Simulate` module that we will produce with `sed`:


```elm
module Simulate exposing (perform)

import Model exposing (Model)
import Model exposing (Msg)
import SimulatedEffect.Http as Http
import ProgramTest exposing (SimulatedEffect)

perform : Model -> Effect -> SimulatedEffect Msg
perform model effect =
    case effect of
        GetPosts ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostsReceived (Decode.list Post.decoder)
                }
        GetPostComments ->
            Http.get
                { url = "/api/posts"
                , expect = Http.expectJson PostCommentsReceived (Decode.list Post.commentDecoder)
                }
```
So the only changes are:
1. The `module` line at the top
2. We have to add an import so that we can use the `SimulatedEffect` type.
3. We have to change the `Http` import to `SimulatedEffect.Http`, because we alias that we don't need to change any of accesses to that module.
4. Finally any uses of `Cmd Msg` we have to change to `SimulatedEffect Msg` or we could have added a type alias.

And that's it. We could also change the type of the function from `perform` to `simulate` if we really wanted ot.
As promised, this is easily achieveable with a sed script:

```bash
IMPORT_HTTP="s/import Http/import SimulatedEffect.Http as Http/g"
IMPORT_SIMEFFECT="0,/^$/ s/^$/\nimport ProgramTest exposing (SimulatedEffect)/"
REPLACE_CMD="s/Cmd/SimulatedEffect/g"
sed "s/module Perform exposing (perform)/module Simulate exposing (perform)/g;
${IMPORT_HTTP};
${IMPORT_SIMEFFECT};
${REPLACE_CMD}" src/Perform.elm > src/Simulate.elm
```

I put this in a file `run-test.sh` and then also call actually run the tests, so that this module is generated before every run of the tests. It's a simple `sed` script and adds negligible time to the test run time.
I think all the parts are fairly self explanatory the `0,/^$/ s/^$` foo at the start of the `IMPORT_SIMEEFFECT` is basically saying "Replace the first occurrence of a blank line with the follow", because what I replace it with starts with `\n` we retain the blank line.

Of course in a real application, including where I actually use this, the `Perform` module is perhaps broken up into smaller modules. That's okay, you can have a `Requests` module that is translated into a `SimulatedRequests` module, and then do the same import translation in your main `Peform -> Simulate` translation. I even translate a `ports` module into one that isn't a `ports` module but uses `SimulatedEffect.Ports` to created simulated versions of the ports.
