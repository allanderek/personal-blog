---
title: "Builder pattern record update operator"
tags: elm syntax 
date: 2021-01-02
---


The *builder pattern* seems to be prettty popular in Elm, [here is an elm-radio episode dedicated to it](https://elm-radio.com/episode/builder-pattern/). 

## The builder pattern

To be clear, the builder pattern is used where you have a data structure, usually a record, which allows for a
reasonable default and can be customised by *chaining* modifier functions. A simple example is best to describe this. Suppose you have *number inputs* in your application. You might have some module tasked with rendering a number input, but the number inputs themselves can have various settings, so you have a `NumberInputConfig` type which controls both the view and update of a number input.

```
type alias NumberInputConfig =
    { max : Just Int
    , min : Just Int
    , required : Bool
    }

default : NumberInputConfig
default =
    { max = Nothing
    , min = Nothing
    , required = True
    }
```

Now you can provide consumers of this with some modifier functions to change the default:
```
withMax : Maybe Int -> NumberInputConfig -> NumberInputConfig
withMax mI config =
    { config | max = mI }

withMin : Maybe Int -> NumberInputConfig -> NumberInputConfig
withMin mI config =
    { config | min = mI }

withRequired : Bool -> NumberInputConfig -> NumberInputConfig
withRequired required config =
    { config | required = required }
```

It seems to be the convention to name these function as `with<Property>`. Though note that these function need not specifically match the exact properties, and they need not take the value of the property, for example:

```
withOptional : NumberInputConfig -> NumberInputConfig
withOptional config =
    { config | required = False }
```

So consumers can build up their configurations using these modifier functions, for example: (assuming the above definitions are in a module imported as `NumberInputConfig`):
```
myConfig : NumberInputConfig
myConfig =
    NumberInputConfig.default
        |> NumberInputConfig.withMax (Just 10)
        |> NumberInputConfig.withRequired False
```

Finally, note that even if your configuration is similar/identical to the default you might wish to use the modifier functions as documentation and also defensively against an update to the default. So to be clear that your number input is required you might do:

```
myConfig : NumberInputConfig
myConfig =
    NumberInputConfig.default
        |> NumberInputConfig.withRequired True
```



## Convenience

The major advantage of this, is if you add a property to the configuration you do not need to change all of your configuration values. So for example we might add a property for 'units':

```
type alias NumberInputConfig =
    { max : Just Int
    , min : Just Int
    , required : Bool
    , units : Maybe Units
    }
type Units
    = Kilograms
    | Kilometers
default : NumberInputConfig
default =
    { max = Nothing
    , min = Nothing
    , required = True
    , units = Nothing
    }
withUnits : Maybe Units -> NumberInputConfig -> NumberInputConfig
withUnits mUnits config =
    { config | units = mUnits }
```

Now, presumably somewhere you are creating a new configuration that uses this new option. However all the existing ones do not need to be updated.

## Disadvantage

The disadvantage of this is what if you *do* want to revise all existing configurations. If all existing configurations were just defined in terms of the record type itself, then the compiler would force you to manually visit each definition and make a decision as to what the new configuration option should be.


## Syntax Noise

Something unsatisfying about this is that there is a lot of noise involved in writing all of the `with<Property>` modifier functions. For the most part these are just boiler-plate that **could** be automatically generated. But generated or not they are just **noise** that distracts from the important part which is the definition of the configuration type, and perhaps some real functions that perform important operations on it.

Something that is frequently proposed is the addition of a chainable record update syntax. Currently Elm allows one to write `.field` which is a derived form (or syntatic sugar if you prefer) for a lambda that accesses that named field: `\r -> r.field`. So the frequent suggestion is to add something like `!field x` which would be a short form for `\x r -> { r | field = x }`. This would obviate the need for most of the `with<Property>` functions.
Your configuations would look like:


```
myConfig : NumberInputConfig
myConfig =
    NumberInputConfig.default
        |> !required True
        |> !max (Just 10)
```

Personally I quite like this.

## Conclusion

Because the advantages and disadvantages of the builder pattern are so complementary, it's difficult to say whether it is a good idea to promote the builder pattern or not. However, the noise it produces suggests that the way to promote it would be to introduce a chainable record update syntax. Given that Elm tends to be pretty conservative about adding new syntax I doubt this would happen. Perhaps that's the right thing, since it's so unclear whether or not the  builder pattern should indeed be promoted.




