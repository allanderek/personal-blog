---
title: "Foldl and foldr"
tags: Elm functional  programming
date: 2021-02-11
---

This is a pretty entry level post on functional programming. It concerns the folding patterns in functional programming. I'll be using Elm as an example language, but the ideas are broadly similar in other functional languages. So I'm talking about the fold functions `List.foldl` and `List.foldr`. I'll talk about a strict/eager language, though one of the interesting things about folds is that in strict languages the `foldl` is the *'good'* one, whilst in lazy languages the `foldr` function is the *'good'* one. I'm going to try to explain why the `foldl` is the good one in strict languages, I'll leave why 'foldr' is the good one for lazy languages for another day.

## What are the folds

There are many descriptions of this available with a simple search. However, the idea is to repeatedly apply a function to each element in a list, and the result of the previous application of the function. I find it pretty intuitive to think of 'building' up some kind of data structure. For example, perhaps re-implementing `Dict.fromList`, that is, building a dictionary from a list of key-value pairs.  So first of all the type of both `foldl` and `foldr`:

```elm
List.foldl : (a -> b -> b) -> b -> List a -> b
List.foldr : (a -> b -> b) -> b -> List a -> b
```

The first argument is our function to be repeatedly applied, the second argument is an initial starting value, and the third argument is the list of items to repeatedly apply the function to. The type of the function we wish to create:

```elm
fromList : List (key, value) -> Dict key value
```

Whilst I'm a massive fan of polymorphism, sometimes it is easier to reason about functions by just making the types more concreate, so lets consider making this function strictly for a dictionary which maps strings to integers, and hence we start with a list string-int pairs.


```elm
fromList : List (String, Int) -> Dict String Int
```

So this means that our two *fold* functions have the instantiated types:

```elm
List.foldl : ((String, Int) -> Dict String Int -> Dict String Int) -> Dict String Int -> List (String, Int) -> Dict String Int
List.foldr : ((String, Int) -> Dict String Int -> Dict String Int) -> Dict String Int -> List (String, Int) -> Dict String Int
```

So let's fill in the details:


```elm
fromList : List (String, Int) -> Dict String Int
fromList keyValues =
    let
        insertPair : (String, Int) -> Dict String Int -> Dict String Int
        insertPair (key, value) dict =
            Dict.insert key value dict
    in
    List.foldl insertPair Dict.empty keyValues
```

The same thing works for `List.foldr`. To see how this works, imagine calling this function with a list of exactly one pair, and then a list of exactly two pairs.

## The difference between foldl and foldr

What happens if you use our `Dict.fromList` to create a dictionary from a list of key-value pairs in which we have a contractitory set of pairs. for example: `[ ("a", 1), ("a", 2 ) ]`. Well, obviously we'll only have one `"a"` key in the dictionary, but what will it be mapped to? If you use `foldl` to write `fromList` then:

```elm
d = fromList [ ("a", 1), ("a", 2) ]
Dict.get "a" d
> Just 2
```

However, if you use `foldr` you will get `Just 1` as the answer. Why? Because left fold, applies the values in the list from the left first, so in this example what you effectively get is:

```elm
Dict.insert "a" 2 (Dict.insert "a" 1 Dict.empty)
```

For some this is a bit clearer using the right pizza `|>` operator, if you're not comfortable with that just ignore this code segment:


```elm
Dict.insert "a" 1 Dict.empty
    |> Dict.insert "a" 2 
```

Either way you view it, it is clear that the insertion of the mapping from `"a"` to `2` is being inserted to the dictionary that already includes the mapping from `"a"` to `1` and therefore **replaces** that mapping. In this case whichever mapping is inserted into the empty dictionary is being inserted first, and will therefore be replaced when the other mapping is inserted. Because `foldl` applies items from the left first, the left most mapping is inserted into the empty dictionary and is thus replaced by the later mapping.

The `foldr` function applies elements of the list starting from the right:

```elm
Dict.insert "a" 1 (Dict.insert "a" 2 Dict.empty)
```


```elm
Dict.insert "a" 2 Dict.empty
    |> Dict.insert "a" 1
```

So in this case it is the latter mapping which is inserted first and therefore which gets replaced.

### Reversing a list with foldl

This is sometimes easier to see if we use the cons operator as our function. The cons operator `(::)` is just a function with the type `a -> List a -> List a` so it can play the part of the function in a fold.

```elm
> List.foldl (::) [] [1,2,3,4,5]
[5,4,3,2,1 ]

> List.foldr (::) [] [1,2,3,4,5]
[1,2,3,4,5]
```

This is because the `foldl` function applies from the left first, so the first thing it does is cons `1` onto the empty list, ie. `1 :: []`, it then takes the next element and cons that on, so `2 :: [1]` and, then the next is consed on so `3 :: [2, 1]` and so on. By contrast `foldr` recurses down to the end of the list first and then applies each element *from the right*, so the first cons `foldr` does is `5 :: []`, and then the second last number is consed on to that list `4 :: [5]` and then the third last element `3 :: [4, 5]`, and so on. So `List.foldl (::)` reverses a list whilst `List.foldr (::)` has no effect.

## Some more folding examples

The result of a fold does not need to be a larger data struction. For example you can return just a number, such as with the `sum` function, and very similarly the `product` function:

```elm
sum : List number -> number
sum l =
    List.foldl (+) 0 l

product : List number -> number
product l =
    List.foldl (*) 1 l
```

You can use a `fold` to search within a list, when you do so, you often need to inspect the list to check that it is not empty first, for example, here are definitions for `minimum` and `maximum`:

```elm
min : number -> number -> number
min x y =
    if x < y then x else y

max : number -> number -> number
max x y =
    if x < y then y else x

minimum : List number -> Maybe number
minimum l =
    case l of
        [] ->
            Nothing
        first :: rest ->
            List.foldl min first rest
                |> Just 
maximum : List number -> Maybe number
maximum l =
    case l of
        [] ->
            Nothing
        first :: rest ->
            List.foldl max first rest
                |> Just 
```

You can also use a `fold` to *'flatten'* a data structure, such as flattening a list of lists into a single list:

```elm
flatten : List (List a) -> List a
flatten listOfLists =
    List.foldr List.append [] listOfLists
```

Of course flatten is in the standard library as `List.concat`. Note that I'm using a `foldr` here, because with a `foldl` the elements would be reversed, an alternative would be to reverse the arguments to `List.append`:


```elm
flatten : List (List a) -> List a
flatten listOfLists =
    let
        flippedAppend left right =
            List.append right left
    in
    List.foldl flippedAppend [] listOfLists
```


## Which should I use?

So the first thing is to decide which would give you the correct answer. The example above with the dictionary showed that you get two different answers depending on which you use. If you were doing this in a program would you want the key-value pairs at the start of the list to take precedence over those at the end of the list? If so, then use `foldr` but if not, then use `foldl`. But some of the examples we've seen are order independent. Such as `sum`, `product`, `maximum` and `minimum`. In a **strict** language such as Elm, if the order of applications does not matter, then you probably want to use `foldl`. Why?

This is to do with something called *'tail-recursion'*. A *'tail-recursive'* function is a function that is recursive, but that the *last* thing we do in the recursive case is call the function recursively. Because of this property the compiler can essentially turn your recursive function into a loop, which is faster than a set of recursive calls. Here is a tail-recursive function:

```elm
isEven : Int -> Bool
isEven i =
    if i < 0
    then isEven (abs i)
    else case i of
        0 ->
            True
        1 -> 
            False
        _ ->
            isEven (i - 2)
```

Note in both of the recursive calls, it is the very last thing that the function does before it returns. The key point is that you can just *pretend* that the recursive call is **actually** the original call, because when you return from the recursive call you are returning the same value from the current call. Here is a function that is **not** tail-recursive:

```elm
length : List a -> Int
length l =
    case l of
        [] ->
            0
        _ :: rest ->
            1 + (length rest)
```

Notice how **after** the recursive call returns we have to modify the answer before returning from the current call, in particular we have to increment it by one. Now it is possible to transform such a function into a tail recursive one, but it's a little clunky:

```elm
length : List a -> Int
length outerList =
    let
        lengthAux soFar l =
            case l of
                [] ->
                    soFar
                _ :: rest ->
                    lengthAux (soFar + 1) rest
    in
    length 0 outerList
```

Notice how now, the recursive call to `lengthAux` is indeed the very last thing that `lengthAux` does. So the `lengthAux` function is tail recursive. So now that we know that, lets look at the definitions for `foldl` and `foldr`, first from the left:

```elm
foldl : (a -> b -> b) -> b -> List a -> b
foldl f b elements =
    case elements of
        [] ->
            b
        (x :: xs) ->
            foldl f (f x b) xs
```

Great, you see the last thing the function does before returning is make the recursive call. So when the recursive call returns we just return that value unchanged, hence this function is tail-recursive and can be optimised. Now `foldr`:


```elm
foldr : (a -> b -> b) -> b -> List a -> b
foldr f b elements =
    case elements of
        [] ->
            b
        (x :: xs) ->
            f x (foldr f b xs)
```

Ach, here after we recurse, we have to apply the `f` function to the head of the list and the result of the recursive call, so `foldr` is not tail recursive. This function cannot be transformed in the same way that `length` could be. Ultimately you need to maintain a stack of the elements that you will later apply to the function.

One final point, I've seen code that either transforms a non-tail-recursive function into a tail-recursive one, or does something awkward in order to call `List.foldl` rather than `List.foldr`. In either case it might be the right thing to do **if** your function is likely to be called on a list with a large number of elements. If however you're writing a function that operates say on the list of children of a given person, then that list is unlikely to exceed double digits in length, hence your *'optimisation'* is likely just both slower and more complicated for someone to comprehend. If on the other hand your list represents say the characters in a source code file, or pixels in an image, then making any functions that operate over it tail recursive may well be a major boon for performance, particularly in the cases you care about (eg. larger files).


For a lazy language, if the order does not matter then you probably want to use `foldr`, for reasons that are slightly beyond the scope of this post.


