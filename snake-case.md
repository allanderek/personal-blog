---
title: Camel case vs snake case
description: Musings on my extremely mild preference for snake-case over camel-case
date: 2021-01-01
---

**Update**: A couple of experiments that seem to give contradicting results for which case style is better for actually recognising identifier names: [Camel > Snake](https://chuniversiteit.nl/papers/why-camel-case-is-better-than-snake-case) and [Snake > Camel](https://chuniversiteit.nl/papers/why-snake-case-is-better-than-camel-case).

Often when naming entities whilst programming we are forced to combine several words into a single lexical token.
So instead of writing `my function` we have to call it `my_function`(snake case) or `myFunction`(camel case), or
perhaps some other contortion, but these are the two most common.

Elm and Haskell both use, by convention, camel case.  It's pretty clear that you should use whatever the convention is for the language you're writing code in. However, given the choice, I would choose snake-case. Here are the very minor points that I think give snake-case the advantage over camel-case. 


### Abbreviations

Often, you need to include some kind of abbrievation in your name, such as `HTTP`, where that abbreviation is typically written in upper case. I feel this clashes with camel-case.  Suppose you need to make a name out of a phrase such as "send HTTP request", in snake-case this translates pretty well as `send_HTTP_request` whilst in camel-case you would have to write either `sendHTTPRequest` or `sendHttpRequest`.

Neither of these options for camel-case seem great to me, but more importantly you have to make a choice, this means there is scope for different people to make different choices, which leads to inconsistency.


### Editor support

I have an `nvim` plug-in that allows me to operate on camel-case 'words' so for example if I have the cursor at the start of the word `makeRequest` and I type `d\w` this will delete 'make', rather than the whole 'word', and I can type `c\wsend` to change it from `makeRequest` to `sendRequest`.  This is quite neat. I can still operate on the whole word as well `dw` would delete the whole `makeRequest`.
 
However, using snake-case, I don't need a plugin, this just works, admittedly I need to use `dW` to delete the whole `make_request` and the `make_request` might be part of a 'chain' such as `make_request.foo.bar` where I do not wish to delete after the `.`, but I can affect this with `dt.`. 

There are arguments both for and against the general solution here, but the point is, that with snake-case a reasonable solution just works out of the box, whilst camel-case requires a plug-in to even get started. Similarly in non-vim-based editors 'Ctrl+arrow' works pretty conveniently for snake-case and less so (in my opinion) for camel-case. 

### Other Capital Meanings

Many languages distinguish the meaning of a word by the case of its starting letter. In Elm, modules, constructor, and type names all start with an upper case letter, whilst value names being with a lower case. In Go, if the first letter is uppercase the name is exported and not-exported otherwise. Whilst using camel-case doesn't prevent any of this (clearly since the convention in Elm is to use camel-case), it does mean you have two meanings of case in the same word. It's easy enough to get used to, but why?

### Qualifying 

Occassionally you want to do something like qualify a name or unqualify a name. So you might want to turn something like `request` into `mainRequest`. In camel-case this involves changing a letter, but in snake-case (`main_request`) it involves only adding letters. Similarly you might be moving something like `loginRequest` into a more general function in which it is just a `request` and again making that change in camel-case involves changing a letter whilst in snake-case involves merely deleting some characters.


## Conclusion

So those are a couple of very minor points in favour of snake-case and I cannot think of any particular reason to use camel-case. Perhaps the underscore is inconvenient for some?  I guess snake-case does mean that your names are longer. Anyway in general I find I can get accustomed to pretty much any (sane) syntax, even if I find it awkward to begin with. However, if I had the choice (and that would pretty much only been when designing my own language), then I'd opt for snake-case.
