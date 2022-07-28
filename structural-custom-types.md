---
title: Structural Custom Types
tags: elm gren programming
date: 2022-06-29
---

There have been a few proposals for extensible custom types in Elm, the idea is that they are somewhat analogous to extensible record types. In this post I wish to give a proposal for how to make custom types 'extensible' that could play well with opaque types. The key point is that we need not focus on extensibility so much as the distinction between structural and nominal types.

The slightly longer summary is to say that we can make custom union types structural types. This means that custom union type declarations are actually type aliases, just as record type definitions are in Elm. Two record types can share the same field, and thus it is trivial to make two record types one of which is a sub-type of the other. Similarly if custom union types are **structural**, and their associated definitions type aliases, two such custom union types can share a constructor, and again it is trivial to create two custom union type definitions one of which is a sub-type of the other. With this apparatus we can implement opaque types by choosing to expose a type either nominally or structurally, this gives us opaque types in which the underlying type can be a primitive type, a function type, a record type or a custom union type.

## Structural types

Structural types refers to types in which the type checker looks at the structure of two types to determine whether they are compatible. For example we might say:

```elm
type alias Email =
    String


emailLength : Email -> Int
emailLength email =
    String.length email
```

This works, because a `type alias` in Elm doesn't define a new type, it's just a way to refer to the original type. So the Elm type checker can see the structure of the type of the value `email` is `String` even though its type is given as `Email`. Because it can see the structure of this type, it knows that it is type-safe to pass `email` in as the first argument to `String.length : String -> Int`. 

This works with record types as well, because those are also type aliases. What matters is the structure of the type, not the name that you have given to it. So we can do:

```elm
type alias Child =
    { name : String
    , age : Int
    }
type alias Parent =
    { name : String
    , age : Int
    }

areTheyTheSame : Parent -> Child -> Bool
areTheyTheSame parent child =
    parent == child
```

Again this works, because although we have given different named types to the two arguments, they are structurally the same.


## Nominal types

In Elm, custom union types are nominal. Creating a custom union type essentially gives you constructors to create a new type with the given name. There is no other way to create that newly named type. Hence when we wish to check if a value of a given custom union type is compatible, we need only check the name of the type, not its structure. 

In Elm you cannot do the following, because you would be defining two **distinct** constructors with the same name:

```elm
type Handedness
    = Left
    | Right

type Correctness
    = Right
    | Wrong
```

The problem here is that if you used the `Right` constructor, the type-checker would have to guess whether you meant to create a value of type `Handedness` or a value of type `Correctness`.

## Making Custom types structural

The key difference between record and custom types in Elm is that record types are structural, whereas custom types are nominal. Instead of a custom `type` declaration defining constructors, which are essentially functions from the constructor arguments to the nominal custom type, we can instead represent the return type of a constructor to be exactly the singleton custom type of the constructor used. This would mean we could then define multiple functions that could accept such a constructed value.

So, we can construct a value without defining a custom `type`. Here we need some extra syntax to refer to a custom union structural type. I'm going to use square brackets here, this is analogous to using curly brackets for record types. So just as a record type is written as `{ field-name : field-type (, field-name : field-type)* }` we will write a structural custom type using `[ constructor-name (type-argument)* (| constructor-name (type-argument)*)*]` if you are comfortable with pseudo-parser syntax if not the general patterns are:

```
{ field-name1 : field-type1
, field-name2 : field-type2
, ...
}

[ constructor-name1 : constructor-type1a constructor-type1b ...
| constructor-name2 : constructor-type2a constructor-type2b ...
]
```

With that syntax we can define a value with a structural custom union type, the simplest being:

```elm
value : [ Duck ]
value =
    Duck
```

Now we can define a function that accepts some set of constructors:

```elm
isDuck : [ Duck | Swan | Goose ] -> Bool
isDuck bird =
    case bird of
        Duck ->
            True
        Swan ->
            False
        Goose ->
            False

isDuck Duck -- type checks fine.
isDuck Chicken -- type error
```

If we prefer, we can give a `type alias` to the `[ Duck | Swan | Goose ]` type:

```elm
type alias PondBird
    [ Duck
    | Swan
    | Goose
    ]
```

But this has the same effect as a `type alias` for a record type, namely that it is just a convenient way to refer to this type. 

In the same way that you can constrain a polymorphic variant to a more restrictive type than would otherwise be inferred, you could do the same with structural custom union type:

```elm
noIntegers : List Int 
noIntegers =
    []

duck : PondBird
duck =
    Duck
```

Here, `PondBird` is more restrictive than `[Duck]` in the sense that you could pass that value into fewer functions. There are some functions that would accept a value of type `[Duck]` that would not accept a value of type `PondBird`, for example:

```elm
isDuck : [ Duck | Swan ]
isDuck bird =
    case bird of
        Duck -> 
            True
        Swan ->
            False
```

### Sub-typing with structural types

An important difference between structural record types and structural custom union types is the way sub-typing works. A record type `A` is a sub-type of a record-typ `B`, if, for all fields in `B, f : C` there is a field in `A, f : D` such that `D` is a sub-type of `C`. What this means in practice is that a function that accepts type `B` is safe to accept type `A`, because all values of `A` have at least the same shape as `B`, perhaps more which is ignored by the function. A simple concrete example helps to make this a little less abstract:

```elm
sayQuack : { a | quack : String } -> String
sayQuack duck =
    duck.quack

sayQuack { quack = "ACK", age = 2, colour = Brown }
```

The value `{ quack = "ACK", age = 2, colour = Brown }` has additional fields, but that's okay because `sayQuack` doesn't care what **additional** fields the argument has, only that it has the field `quack : String`. It's a slight quirk of Elm that you must explicitly declar the argument type of `sayQuack` as extensible, but that won't concern us in this post.

In contrast, a custom union type `A` is a sub-type of `B`, if for all constructors  in `A, C : D` there is an equivalent one in `B`. Again a simple example helps. 

```elm
speak : [ Dog | Cat | Budgie ] -> String
speak animal =
    case animal of
        Dog ->
            "woof"
        Cat ->
            "miaow"
        Budgie ->
            "tweet"

getPet : Bool -> [ Dog | Cat ]
getPet isDogLover =
    case isDogLover of
        True ->
            Dog
        False ->
            Cat

firstPet : [ Dog | Cat ]
firstPet =
    getPet True

speak firstPet

```

The function `speak` is called with a value of type `[ Dog | Cat ]` which is a sub-type of `[Dog | Cat | Budgie]`. This is perfectly type-safe because the type checker knows each possibility will be handled correctly, but I could not call this with a value of type `[ Dog | Cat | Mouse]` or `[ Dog | Cat | Budgie | Mouse]` or even simply `[ Mouse ]`. Because the case of `Mouse` would not be handled by the function `speak`.


## Exporting, Importing, and opaque types

If we have both record and custom union types represented structurally what does this mean for opaque types? Currently in Elm a custom union type can be exposed using the `(..)` suffix to mean "and all the constructors are exposed". Exporting a type `A` with `A(..)` means that outside the defining module, a value of type `A` can be constructed or deconstructed/inpsected. Sometimes you want only the defining module to be able to create or inspect (ie. constructor or deconstruct/pattern match), in this way you can ensure some property/properties hold true of all values of `A`. For example you might define a value of type `Email` and then only allow the construction of such values via a function that checks whether the given string is indeed a valid email address. Once you do that you know all values of type `Email` do indeed contain a valid email address. This pattern is known as *opaque* types.

I would like to propose that we can expose a type **nominally** or **structurally**. If a type is exposed **structurally** then any code outside of that module can create or inspect a value of that type. In particular the type checker will 'know' the structure of that type outside of its defining module, and can type check code knowing the structure of that type. However, if a type is exposed **nominally** then it's not possible for code outside of that module to know the structure of that type at all. Therefore the only way to create or inspect a value of that type is via the module in which it is defined.

This means that we gain opaque types, not just for custom union types, but for record types and primitive types as well. For example, you could have a module:

```elm
module Email exposing (Email, create, format)

type alias Email = String

create : String -> Maybe Email
create s =
    -- This works inside this module, because inside this module, 
    -- the Email type is **structural**. 
    -- But you couldn't do this outide this module, 
    -- because there the Email type is **nominal** 
    -- and so the type-checker has no way of unifying the String type of 's' 
    -- with the 'Email' in the return type.
    -- Obviously we could do stronger validation
    case String.contains "@" s of
        True ->
            Just s
        False ->
            Nothing

format : Email -> String
format email =
    -- Similarly this works inside this module but not outside it.
    s
```

Now the nice thing is that we could change the representation of `Email`, to be a record type:


```elm
module Email exposing (Email, create, format)

type alias Email = 
    { username : String
    , domain : String
    }

create : String -> Maybe Email
create s =
    case String.split "@" s of
        [ left, right ] ->
            Just
                { username = left
                , domain = right
                }
        _ ->
            Nothing


format : Email -> String
format email =
    String.concat [ email.username, "@", email.domain ]
```

The great thing about this, in contrast with an Elm module, would be that outside of this module, a value of
type `Email` could not be created or inspected, because outside this module the `Email` type is **nominal**. That is true whether the underyling type is a `String` or a record type, because we are choosing to export the **nominally**.

Finally we can do the same with a custom union type:


```elm
module Email exposing (Email, create, format)

type alias Email = 
    [ Email username domain ]

create : String -> Maybe Email
create s =
    case String.split "@" s of
        [ left, right ] ->
            Just (Email username domain)
        _ ->
            Nothing


format : Email -> String
format email =
    case email of
        Email username domain ->
        String.concat [ username, "@", domain ]
```

Again, because the `Email` type is exposed nominally, there is no way to create or inspect a value of type `Email` outside of this module.

We can represent exporting/importing nominally/structurally using the existing Elm syntax, or come up with new keywords or syntax, for example we could use:

```elm
module Email (Email) -- means expose 'Email' nominally
module Email (Email(..)) -- means exposing 'Email' structurally
```
Or we could simply use two new keywords

```elm
module Email (Email nominally) -- means expose 'Email' nominally
module Email (Email structurally) -- means exposing 'Email' structurally
```

Or a single new keyword with a default:

```elm
module Email (Email) -- means expose 'Email' nominally
module Email (Email structurally) -- means exposing 'Email' structurally
```

Many other possible syntaxes are available, this would be strong bike-shed material.

## Is this enough for extensibility

Note that we haven't defined any syntax for extending a custom union type. But just as structural record types can contain the same fields, so could structural custom union types contain the same constructors.

```elm
type alias BusinessDay =
    [ Monday | Tuesday | Wednesday | Thursday | Friday ]

type alias WeekDay =
    [ Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday ]
```

Any function defined over `Weekday` would accept a value of type `BusinessDay` since `BusinessDay` is a sub-type of `Weekday`.

It isn't required, but we could of course imagine some kind of extension syntax:

```elm

type alias BusinessDay =
    [ Monday | Tuesday | Wednesday | Thursday | Friday ]

type alias WeekDay extends BusinessDay =
    [ Saturday | Sunday ]
```



## Catch-all patterns

There is a slight issue with the catch-all pattern `_ ->` related to the way in which a case expression would be typed.

A case expression such as:

```elm
available day =
    case day of
        Monday ->
            True
        Tuesday ->
            True
        Wednesday ->
            True
        Thursday ->
            True
        Friday ->
            True
```

Cannot have the **inferred** type of `BusinessDay -> Bool`, because those five constructors could be defined in multiple custom union types, and in any case custom union types would only be aliases. Instead this would type check as: `[ Monday | Tuesday | Wednesday | Thursday | Friday] -> Bool`. You can of course give a type signature.

Now, what about if we used the catch-all pattern?


```elm
available day =
    case day of
        Monday ->
            True
        Tuesday ->
            True
        Wednesday ->
            True
        _ ->
            False
```

Now we need some new type syntax to write down this type. This is something like:
`[ Monday | Tuesday | Wednesday | a ] -> Bool` where `a` here stands for "any other set of constructors that don't contain 'Monday', 'Tuesday', or 'Wednesday'". Note that we cannot write this as simply `a -> Bool` because the value must be a custom union type, **and** it won't accept a value of say `Wednesday Int`. Note though that here we can simply add a type signature to get the type that perhaps is really wanted:

```elm
available : BusinessDay -> Bool
available day =
    case day of
        Monday ->
            True
        Tuesday ->
            True
        Wednesday ->
            True
        _ ->
            False
```

This because `BusinessDay` is a synonym for `[ Monday | Tuesday | Wednesday | Thursday | Friday]` and that is a more restrictive type than `[ Monday | Tuesday | Wednesday | a]`.

## Exhaustive pattern testing

Treating custom union types in this way means that we do not get exhaustive pattern testing. Because the case expression itself is always exhaustive for some custom union type. 

Of course exhaustive pattern testing remains unaffected for primitive types, such as String and Int.
So for example:

```elm
available day =
    case day of
        Monday ->
            True
        Tuesday ->
            True
        Wednesday ->
            True
```

Is a perfectly valid function. It has type `[ Monday | Tuesday | Wednesday ] -> Bool`. This is all fine, if we want to make sure that our function can accept any `BusinessDay` value, then we should add a type signature:


```elm
available : BusinessDay -> Bool
available day =
    case day of
        Monday ->
            True
        Tuesday ->
            True
        Wednesday ->
            True
```

In which case we would not get a non-exhaustive pattern match failure *per se*, but we would get a type error stating that the type of the expression `[ Monday | Tuesday | Wednesay ] -> Bool` is not compatible with the signature type `BusinessDay -> Bool`. The error message could explain why.

Alternatively the signature could be omitted in which case it is a perfectly valid function, but we could get an error if we ever attempted to call this function with a value that might be of a `Thursday` or `Friday` constructed value. If we never do that, then there is nothing wrong.

## Recursive Extensive Types

This is perfectly enough for recursive extensive types. We can define a simple expression syntax for doing integer arithmetic. The `Apply` constructor takes the name of a function, such as `sin` or `add` and applies that to the list of argument expressions given.

```elm
type Expr
    = IntLiteral Int
    | Apply String (List Expr)
```

Now we could define a slightly more complicated expression language to parse into:


```elm
type ParseExpr
    = IntLiteral Int
    | Apply String (List ParseExpr)
    | Binop ParseExpr String ParseExpr
```

Now we can defined a `simplify` function which converts a parsed expression into the simplier expression kind:

```elm
simplify : ParseExpr -> Expr
simplify parsed =
    case parsed of
        IntLiteral i ->
            IntLiteral i
        Apply name args ->
            List.map simplify args
                |> Apply name
        Binop left name right ->
            Apply name [ simplify left, simplify right ]

```
Side note, it would be **possible** but **difficult** for the type-checker to allow `IntLiteral _ -> parsed` here.

You could already do this with normal nominal custom types in Elm. You would have to give the two expression types different constructors which you could do either by giving them explicitly different names, or by defining each type in a different module. However with this approach you can define a function that operates over both expression types, such as:

```elm
usedFunctions : ParseExpr -> Set String
usedFunctions expr =
    case expr of
        IntLiteral _ ->
            Set.empty
        Apply name args ->
            List.map usedFunctions args
                |> List.foldl Set.union Set.empty
                |> Set.insert name
        Binop left name right ->
            Set.singleton name
                |> Set.union (usedFunctions left)
                |> Set.union (usedFunction right)
```

Because `Expr` is a sub-type of `ParseExpr` you can call `usedFunctions` with a value of type `Expr`.

## Possible drawbacks

### Same constructor in two types

One could imagine defining two types as such:

```elm
type alias Handedness =
    [ Left | Right ]
type alias Correctness =
    [ Right | Wrong ]
```

This could lead to a situation in which you incorrectly passed a value that you intended to be of type `Handedness` into a function that expects a value of type `Correctness`. Note however, that although you intended the type to be of `Handedness` it would only pass type checking if it was really of type `[Right]`. 

The implications of this would need to be considered quite thoroughly.

### Compilation

At the moment custom union types can be compiled using integers to represent the tags, and the compilation can still happen locally. If you wish to use integers for tags, when a tag can be used in many different types (including the strutural argument types to functions), then there would have to be a whole program optimisation/transformation, which gives a unique integer to each tag.
