---
title: "Mutation typing - a not-yet workable idea"
tags: programming functional
date: 2021-02-18
---


I think [mutation testing](https://wiki.c2.com/?MutationTesting) is a pretty fine idea. The idea of mutation testing is to deliberately add a bug into the program and then check that the test suite catches that bug. We add the bug into the program by mutating a part of it, since if you **change** code that you believe is all correct, then presumably it cannot still be correct. Of course mutating the program can be mechanised and hence we can test many many mutant programs. Mutation testing does not test your **program** code, but how effective your test-suite is. Mutation testing is used to improve your test-suite.

In a statically typed language, you only make mutations that retain the types of the program, since otherwise, the program will not compile and you won't be able to run the test suite on it. I had an idea that we could use mutation testing for free, that is even if you do not have a (comprehensive) test suite, or in **addition** to testing your test-suite to also examine how well you are utilising the type-system in your code. 

The idea was pretty simple, if we write a mutatator for our (statically typed) language, and then allow it to do its thing, for each mutated program we check if it still compiles, if it does, then there is perhaps an opportunity to utilise the type system a bit better. The sorts of things we could catch here are places where using a boolean is not the correct thing and you should have a more unique custom type.

The mutator for this scheme would have to be less aggressive, because swapping two expressions which must be the same type will obviously yield a program that continues to type-check. For example swapping the `then` and `else` branches of a conditional expression would always yield a program that type-checks  (assuming the original one did).

So I do not think this scheme would actually work, but I'd like to describe the sort of scenario I was thinking of anyway. Here is an example scenario:

```elm
import Html exposing (Html)
import Html.Attributes as Attributes 

type alias Color =
    String
type alias FontSize =
    String

type alias Name =
    String

viewName : Color -> FontSize -> Name -> Html msg
viewName color fontSize name =
    Html.span
        [ Attributes.style "color" color
        , Attributes.style "font-size" fontSize
        ]
        [ Html.text name ]

... 
    viewName "green" "0.8em" user.name
...
```

One thing mutation testing could do is swap the two arguments about:

```elm
... 
    viewName "0.8em" "green" user.name
...
```

This would obviously introduce a bug, but the type system would not catch it. Would the test suite catch it? Possibly. So traditional mutation testing would do well here, either in showing that your test suite is working or that you are not sufficiently testing this part of your program. However, even without the test suite just the fact that with the mutation the program still compiles is interesting. We could do a lot better with our types here:


```elm
import Html exposing (Html)
import Html.Attributes as Attributes 

type Color
    = Green
    | Red
    | Yellow
type FontSize 
    = ByEm Float
    | ByPx Float

colorAttribute : Color -> Attribute msg
colorAttribute color =
    let
        colorValue = 
            case color of
                Green ->
                    "green"
                Red ->
                    "red"
                Yellow ->
                    "yellow"
    in
    Attributes.style "color" colorValue

fontSizeAttribute : FontSize -> Attribute msg
fontSizeAttribute fontSize =
    let
        fontSizeValue =
            case fontSize  of
                ByEm em ->
                    String.append (String.fromFloat em ("em")
                ByPx px ->
                    String.append (String.fromFloat px ("px")
    in
    Attributes.style "font-size" fontSizeValue

type alias Name =
    String

viewName : Color -> FontSize -> Name -> Html msg
viewName color fontSize name =
    Html.span
        [ Attributes.style "color" color
        , Attributes.style "font-size" fontSize
        ]
        [ Html.text name ]

... 
    viewName Green (ByEm 0.8) user.name
...
```

This is a little more verbose to be sure, but now the same kind of mutation would introduce a type-error:

```elm
... 
    viewName (ByEm 0.8) Green user.name
...
```

There is a problem though. We can all look at this code and realise that it is more safe than the previous code, it's more defensive, we're some how less likely to introduce a bug. However, from the mutation tester's point of view, you haven't really improved the code. It might just as well try the following mutation:


```elm
colorAttribute : Color -> Attribute msg
colorAttribute color =
    let
        colorValue = 
            case color of
                Green ->
                    "red"
                Red ->
                    "green"
                Yellow ->
                    "yellow"
    in
    Attributes.style "color" colorValue
```

Here, all I've done is swap the expression results for `Green` and `Red` around. From the mutator's point of view this is just as valid a mutation, but it wouldn't introduce a type-error, and so the proposed system would conclude, just as before, that you could better type your program.  However, of course this case is just the same as the conditional expression case I gave above. If you swap two branches of a `case-expression` then because the type-checker insisted that they be compatible types in the first case the mutated program must also type-check. Because of this, this kind of mutation would not be used in the proposed scheme. So my claim above that this is just as valid a mutation is false. But does that mean we just leave it out far too many potential mutations? 

Another point here is that you do not really need to go to the bother of doing the mutation and type-checking it, you could just introduce an [elm-review](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/) rule that warns you about two arguments with the same type. Though of course sometimes this is simply unavoidable, eg. `List.append` is just a reasonable thing to do. Though you could only warn in the case that there are more than two arguments to the function.

All-in-all I cannot really see how to make this work. It feels like there are programs that have better types than other programs, hence the advice to try to *'make impossible states impossible'*. It sort of feels like mutating the program and then checking if it still type-checks would help with this, but ultimately I cannot really see how to make it work. It seems like any such scheme will always end up with far too many false positives to be useful, and/or can just be implemented directly in term of rules for warnings rather than rules for the mutations. Perhaps the only cases it will catch will be the ones in which you have multiple arguments to a function of the same type. Perhaps that's a useful thing to check for, perhaps not.
