---
title: "Another polymorphism blind spot"
tags: programming  elm
date: 2021-04-02
---

Yesterday I [detailed](/posts/2021-04-01-polymorphism-blind-spot) a simple fragment of code that is untyped in traditional Hindley-Milner type systems (such as that employed by Elm), but which is a perfectly reasonable bit of code. I think of this as a minor irritation and quite easy to live with. I term it a blind-spot of polymorphism, because in general I find that polymorphic Hindley-Milner style type systems map pretty well on to the types of programs that I *wish* to write. Aside from the issue of meta-programming, but that's quite a separate issue.

Anyway, today there is another blind-spot and this is more than a minor-irritation, although it doesn't come up all that **often**.

Suppose you're writing a code editor, you can have different kinds of buffer representing different kinds of source code. Each different kind has its own *mode*. So let's suppose you have an `Elm` kind of buffer and a `Json` type of buffer, and those has related states, and also related modes. How do you represent the list of open buffers?

So first we might define an 'Elm buffer' and a 'Json buffer':

```elm
type alias ElmBuffer =
    { text : String
    , definedNames : List String
    }

type alias JsonBuffer =
    { text : String
    , indentation : Int
    }
```

Then the modes, obviously there would be a lot more 

```elm
type alias Mode a  =
    { format : a -> a
    , highlight : a -> [( Int, Color) ]
    }
type alias ElmMode =
    Mode ElmBuffer

type alias JsonMode =
    Mode JsonMode
```


So we would like to represent the list of open buffers as a list of buffers and their associated modes. But you cannot do this, even though you would like to be able to write the type:

```elm
type Buffers =
    List (a, Mode a)
```

Unfortunately if you try this, you will have the problem that Elm does not allow a *'free'* type variable. Note that there is a missing type variable between `Buffers` and `=`, so you have to write:


```diff
-- type Buffers =
+ type Buffers a =
    List (a, Mode a)
```

But this of course means that all the buffers must be of the *same* type. So I cannot write a function such:

```elm
formatBuffers : List (a, Mode a) -> List (a, Mode a)
formatBuffers buffers =
    let
        formatBuffer (b, m) =
            m.format b
    in
    List.map formatBuffer buffers
```

The Hindley-Milner system can type this, but the input must be a list of uniform buffers. Even though we can see that giving a list of heterogenous buffers would be perfectly fine, because we only ever apply a mode's format function to its own buffer.

Now, you can get around this, but it has a significant drawback:

```elm
type Buffer =
    Elm ElmBuffer
    Json JsonBuffer
```

Then your `formatBuffer` becomes easy enough:


```elm
formatBuffers : List Buffer -> List Buffer
formatBuffers buffers =
    let
        formatBuffer buffer =
            case buffer of
                Elm elmBuffer ->
                    Elm (ElmMode.format elmBuffer)
                Json jsonBuffer ->
                    Json (JsonMode.format jsonBuffer)
    in
    List.map formatBuffer buffers
```

Notice here that we just represent the mode with a module, because we know longer have to carry the *'mode'* information around with us. And that's the major problem. This system is not extensible. If you release the editor as a program that's perhaps expected, but if you release it as a library even then it's not possible for the library consumer to add their own mode. They would have to fork the library (or contribute back to the original one).

So the blind-spot is that you cannot enclose a parcel of data that is internally self-consistent and prevent some of the type being leaked into the broader value. So more generally you cannot write the following:

```elm
zipApply : List (a, a -> Int) -> List Int
zipApply pairs =
    List.map (\(f,a) -> f a) pairs

numbers = zipApply [ (1, identity), (2.0, round) ]
```

Even though, we know there is nothing wrong with it.

