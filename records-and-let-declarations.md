---
title: "Elm records and let declarations"
tags: elm syntax 
date: 2021-01-22
---

Elm is pretty frugal when it comes to syntax. This has many benefits, one oft cited is that it helps beginners get started pretty quickly, in particular after learning the syntax for ten minutes or so a beginner can understand most Elm code out in the wild. I think a more compelling advantage is that the lack of syntax reduces the ways in which the same problem can be solved. It of course doesn't eliminate this entirely, some people like to use `|>` and others much prefer `<|`, or even just to use parentheses.

It is in this context that I've been thinking about let-declarations and record expressions. The two are pretty similar. Both define a bunch of names. There are a few differences

* Obviously a let-expression has an extra expression *after* the set of name definitions
* The name definitions in record expressions cannot take advantage of the syntactic sugar for function definition.
* The name definitions in record expressions cannot be recursive
* The name definitions in record expressions cannot refer to the other names defined therein
* Record expressions can do field update.

So the simple question is, could we relax these differences and perhaps merge the two syntaxes (or whatever plural for syntax your prefer). I don't think so, simply because I think the separate syntax conveys intent well.  However, let's look at the differences in turn.


## Extra expression

Any unified syntax would have to allow for there to be an extra-expression at the end or not. So either you would extend record expressions such as:



```elm
{ x = 1
, y = 2
} in
x + y
```

Or you would allow a *let* declaration block without a finishing `in` expression


```elm
let
    x = 1
    y = 2
```

Would be equivalent to `{ x = 1, y = 2}`.

## Lambda 
Lambdas in record expressions. I think:

```elm
{ f = \x -> x + x
, ...
}
```
could be written as:

```elm
{ f x = x + x
, ...
}
```

I don't see any drawbacks to allowing this, but perhaps I'm missing something?

## Recursive and other names

These are actually the same one, in *let* expressions you can refer to names defined at the top level of the same *let*, including the one you're currently defining. You cannot do that in records. I find that occasionally this results in slightly awkward *let* definitions that are there only so that you can refer to another defined record field:

```elm
let
    elements =
        List.filter cond allElements
in
{ num_elements = List.length elements
, elements = elements
}
```

In this example I only need the *let* declaration because I need to use the defined name `elements` in the definition of `num_elements`. Wouldn't it be a bit nicer if I could just write:


```elm
{ num_elements = List.length elements
, elements = List.filter cond allElements
}
```

Again, I don't really see any disadvantage to allowing this. 

## Field update

In order to merge the two syntaxes, you could allow someway to do this in whatever syntax ended up being the merge syntax, so either record syntax would need a way to do *let-in*:

```elm
{ x = 1 , y = 2} in r.x + r.y
```

Or, *let* without an end expression would need to be allowed to do record update, eg.:

```elm
let r |
    x = 1
```

I guess I would prefer the first. But since I don't think the syntaxes should be merged I don't think we need a solution to this.

## Record expressions that ignore some fields

So obviously you often do something like:


```elm
let
    a = ...
    b = ...
    c = ...
in
{ a = a
, c = c
}
```

If you were to merge the two syntaxes you would need a way to do this in record field, but I guess you could just use the same approach:

```elm
{
    a = ...
    b = ...
    c = ...
}
in
{ a = a
, c = c
}
```

## Conclusion

As I said, I do not think these two syntaxes can be usefully merged, because I think that the two provide useful clues as to the intent of code. Always a useful property.

Nevertheless, I think that record expressions could have their syntax relaxed and brought more in line with `let` declarations. I do not see much problem with doing that. However, if your bar is that you need to find a bunch of code that would be materially improved by such a change I haven't done the homework, and I'm not sure you will find that many bits of code that could be usefully improved. But I could be proved wrong.

## Final thought

Another thought is that you could go the opposite way and introduce things allowed in the *let* declaration that wouldn't be allowed in a record expression, for example *local module imports* or *type declarations*:


```elm
let
    import Html exposing (Attribute)
    type alias Heading =
        { class = Attribute msg
        , title = String
        }
in
...
```
Where the scope of both `Heading` and `Attribute` would be limited to the *let* expression.

