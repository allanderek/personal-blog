---
title: "Type classes are meta-programming"
tags: programming elm
date: 2021-02-02
---


A feature that is semi-regularly requested in Elm, or at least discussed is the issue of type-classes.
Type classes are a means in Haskell of restricting polymorphism, which then allows you to write more generic functions that you would otherwise be able to. In fancy words that means that type classes support ad-hoc polymorphism, but you can forget about that. I'll start off with a simple example, then show how you could acheive a similar  result without the type classes. Finally I'll use this to argue that therefore type-classes are a limited form of meta-programming.

## Example

Suppose you have a list and you wish to extract the 'maximum'. You can easily do this for a list of integers:

```elm
maximum : List Int -> Maybe Int
maximum l =
    case l of
        [] ->
            Nothing
        (head :: rest) ->
            case maximum rest of
                Nothing ->
                    Just head
                Just maxOfRest ->
                    case head > maxOfRest of
                        True ->
                            Just head
                        False ->
                            Just maxOfRest
```

Now, this is a nice function but it doesn't really depend upon much of the properties of integers, so it would be nice to extend this to at least other numbers. Elm actually has limited supports for a small set of standard defined classes, so you can actually do the same, but with a special `number` type parameter instead of `Int`:


```elm
maximum : List number -> Maybe number
maximum l =
    ...
```

Now, suppose you wanted to run this function over arbitrary types, you would be out of luck, because there is no way to compare, say functions, so you wouldn't be able to find the maximum of a list of functions. But you may wish to do something like find the maximum of a list of strings. In Haskell, you can define a type class. So you can do something like the following:

```haskell
class IsGreater a where
  (>) :: a -> a -> Bool
```

This states that a type `a` is a member of the type class `IsGreater` if you define a function with the name `(>)` and the type `a -> a -> Bool`. Now of course you cannot just define such a function you also have to tell the type system that you mean it to be the function used as the *'witness'* that this type is in the type class `IsGreater`. So you could place strings in this type class with the following:

```haskell
instance IsGreater String where
    (>) = \left right -> String.len left > String.len right
```
Of course it is up to you how you define that `(>)` function over strings, but here I've just chosen string length. Now, you can re-write the above `maximum` function with the following type:


```haskell
maximum :: IsGreater a => List a -> Maybe a
```

This type means: *'For all types `a` that are in the type class `IsGreater`: List a -> Maybe a'*.
So in other words it is less restrictive than `List Int -> Maybe Int` but more restrictive than `List a -> Maybe a`. It's basically exactly what we want.


## Type-classless solution

Elm does not have ad-hoc type classes, but you can acheive a similar result. Rather than pass the type-class around at compile-time we can instead pass the *'witness'* around at runtime. In this style we could write our `maximum` function two different ways:

```elm
maximumBy : (a -> Int) -> List a -> Maybe a
maximumBy magnitude l =
    case l of
        [] ->
            Nothing
        (head :: rest) ->
            case maximumBy magnitude rest of
                Nothing ->
                    Just head
                Just maxOfRest ->
                    case magnitude head > magnitude maxOfRest of
                        True ->
                            Just head
                        False ->
                            Just maxOfRest
```

Or you can do `maximumWith` which would correspond to the type class definition given above:


```elm
maximumWith : (a -> a -> Bool) -> List a -> Maybe a
maximumWith compare l =
    case l of
        [] ->
            Nothing
        (head :: rest) ->
            case maximumWith compare rest of
                Nothing ->
                    Just head
                Just maxOfRest ->
                    case compare head maxOfRest of
                        True ->
                            Just head
                        False ->
                            Just maxOfRest
```

There is something of a convention to name these *'witness'* functions `...By` for those that take a single parameter and `...With` for those that take two. See for example: [list-extra](https://package.elm-lang.org/packages/elm-community/list-extra/latest/List-Extra) package documentation.

## Conclusion

I see type-classes as a limited kind of meta-programming. They allow you to essentially create a family of functions from a single definition. This doesn't allow you to do anything you cannot already do without type-classes, but it can make your definitions somewhat cleaner. You can also think of type-classes as doing some [partial evalation](https://en.wikipedia.org/wiki/Partial_evaluation) where you know at compile-time all the *'witness'* parameters you're going to pass in. 

So should type-classes be added to Elm? Type classes are undeniably useful. They are also undeniably an additional concept for beginners to learn. One issue is that type-classes are most useful for generic code, and therefore are most useful in the standard library. Just as un-restrained polmorphism is very useful in the standard library, ad-hoc polmorphism is also very useful for definining arithmetic/comparison based functions such as the `maximum` example in this post. This means that beginners are necessarily thrust into using type-classes immediately.

It's pretty clear that Elm has a stated goal of being friendly to beginners, so perhaps this is a trade-off consistent with Elm's goals. But there is another reason to hold off. If, as I've been vaguely arguing, type-classes are a restricted form of meta-programming, what if some other form of meta-programming is discovered which would be more appropriate for Elm's goals. I do not know how likely this is, but Elm has been consistently conservative in adding new features until all current possibilities have been investigated thoroughly. This thoroughness has proven so far to result in what I consider a very cleanly designed language, with few warts. So I'm quite happy for it.
