---
title: "Surprising Elm"
tags: programming elm 
date: 2021-02-13
---

Popping up on my feed was an [article on learning Python by looking through the gotchas and unexpected things](https://github.com/satwikkansal/wtfpython) that happen in Python. It's a nice enough article, and clearly a lot of time has been spent polishing it off and even publishing a package to accompany it. What is a little disconcerting though is the sheer number of surprising snippets. A pretty reasonable goal for a language would be to have as few surprising parts as possible. That's another advantage of having a very small language, there are fewer things that could have been designed in a surprising way, and perhaps more importantly, there are fewer moving pieces that can **combine** to produce strange or surprising behaviours.

Looking through Elm it's relatively difficult to come up with anything that is strange or surprising. There is of course the odd bug, apparently this code causes an infinite loop in Elm:

```elm
String.right 1 "🙈" |> String.toList
```

Okay so that might be surprising but that's clearly not intended behaviour. What about intended behaviour? Sometimes beginners in Elm are surprised that their `Html.input` values do not change on user input because they have forgotten (or not known) to add an `Html.Events.onInput` handler for the event. They are similarly surprised that they have to manually store the the input value on the `Model` and update it in response to new input:

```elm
Html.input
    [ Attributes.value "" ]
    []
```

Okay that might be surprising behaviour to beginners, but it's certainly consistent with *The Elm architecture*.


Some are a little surprised by the lack of user-definable type-classes, and hence at the choice of standard type classes. In particular there is no `eq` type class, either something is comparable, in which case it can be ordered, or it cannot be checked for equality. This can get you into trouble:

```elm
equality : a -> a -> Bool
equality a b =
    a == b
```

Note that this admits any type for the two arguments, they are not constrained by being either `comparable` or member of some non-existant `eq` type classes. So you can apply this to functions and you will get an error.

Another place where this can surprise people is that you cannot have a dictionary where the keys are just any type. They must be `comparable`, which in particular means that you cannot have custom types as the key. This might be a little surprising but at least it cannot lead to bugs, and there are a couple of (admittedly a little unsatisfactory) solutions to this issue.

Other than that I'm not sure there is much in the language that is surprising. Perhaps the fact that you cannot have anything other than a simple name variable in the first part of a record update expression, or that you cannot define your own infix operators. Therein lies something notable, almost all of the surprises in Elm are things that you surprisingly are not allowed to do. That's a much quicker, and less expensive surprise than a surprise of semantics, such as [Python's mutuable default argument](https://github.com/satwikkansal/wtfpython#-beware-of-default-mutable-arguments). In other words you're less likely to introduce a bug because you didn't know that you couldn't define your own infix operator, than you are because you didn't know that default arguments are evaluated once.


## Conclusion

So although there are a couple of mildly surprising parts of Elm, I think the language design team can be thus far pretty happy with how intuitive, consistent and clean a language Elm is.
