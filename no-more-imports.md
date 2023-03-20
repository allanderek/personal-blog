---
title: No more imports
description: Do we really need imports?
date: 2022-10-06
tags: [elm, gren, programming, program structure]
---

Imports in most languages always seem like a bit of an add-on, a separate language to the actual language.
Today's post is a thought experiment regarding removing imports from Elm/Gren. Ultimately I think it's probably worthwhile
having the import statements there, but this is an interesting (and quick) thought experiment.

If we remove `import` statements from the head of a module file the first thing to note is that a file then becaomes one `module` declaration line followed by simply a list of top level declarations.
However, we would obviously need someway to refer to values and types defined in another module.

Suppose we just say that **all** modules that **could** be referenced are "in scope". I'll define "in scope" shortly
but first we need need to decide what modules could be referenced. Fortunately this is defined pretty strictly
in the elm/gren.json. It's just any module that appears in a source-directory or in package dependency.

To actually use a value from another module, the module has to be "in scope". All that means is that the fully
qualified module name is available to the programmer. It is equivalent to taking all modules that could be referenced
and generating an import statement with no `as` or `exposing` clause. Assuming you have `elm/core` or `gren/core` in
your list of dependencies you will have an implied set of import statements that looks like:

```elm
import Array
import Basics
import Bitwise
import Char
import Debug
etc.
import Platform.Cmd
import Platform.Sub
```

This is already a perfectly usable language. It's a little clunky, particulary with respect to infix operators.
Everything needs to be qualified, and even module names cannot be aliased. So no more referring to `Json.Encode.Value` as `Encode.Value`.

Let's deal with the first of these, everything must be qualified. First of all, most types and functions I [qualify already](/my-import-scheme).
But suppose you're one of those people who do not like to write `Html.div` everywhere. That's fine, you can alias that yourself:

```elm
div : List.List (Html.Attribute msg) -> List.List (Html.Html msg) -> Html.Html msg
div =
  Html.div
```

Okay I'll agree that this is not terribly beautiful, mostly because of all the qualified types. You could get around this by not giving the signature:
```elm
div = Html.div
```

This is now exactly equivalent to having an `exposing` clause when importing the `Html` module.
Except that `elm-review` might complain about it because it is a top-level definition without a type-signature, but the `elm-review` rules for this could be easily ammended to not warn
about alias definition unadorned with a type signature. If you leave the type-signature on you will of course have to update the alias if the aliased value changes in type (more likely for values defined in your own modules).
In any case, I feel that such aliases, at least for values, should be used sparingly.


But let's worry about those type definitions, you can of course do the same thing and just alias those:

```elm
type alias Html = Html.Html
type alias List = List.List
```

You cannot simulate an `exposing (..)` clause, but I definitely feel that those are dubious anyway. I could certainly live without them.


That means we can simulate any `exposing` clause of a module import, other than `(..)`.
I tend to qualify even constructor names, but we would also lack a means of opening a custom union type.

The last thing that would not be possible is simulating an `as` clause. If we allowed a module as a value, then we could do:

```elm
Html.Attributes = Attributes
```

With this, we would have no need for import statements.

The fact that you would need to provide some way to alias a module name to a shorter one, might open up some nice possibilities, most obvious of which is providing a very short alias to a module that is only used within on top-level definition (presumably because it uses that module extensively)
So for example you might generally wish to refer to it as `Html.Attributes`, but may when defining some element with a lot of attributes you could do:

```elm
let
    A = Html.Attributes
in
Html.button
  [ A.id ...
  , A.class ...
  , A.disabled ..
  .. etc.
  ]
```

### Advantages

You can have any of these aliases at any point in the module, including within a `let-in` expression thereby constricting its scope.

Module imports don't need to be kept up to date, and you can just use a value from another module without going to the top of the file and adding an `import` statement (or checking if one already exists).

I like that you would be forced into making module aliases a little more expressive.

### Disadvantages

It's more difficult to see at a glance which modules are imported by a module.

You cannot, as easily, expose all the constructors of an imported custom union datatype.

## Conclusion

Mostly imports then are a set of syntactic sugar forms for declarations.
On balance I think it's probably worthwhile having those syntactic sugar forms, but not by much.
