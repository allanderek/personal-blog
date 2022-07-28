---
title: "Elm missing features #1033"
tags: programming elm
date: 2021-03-31
---

Global variables have a bad name, mostly for good reason, but as with most things which are *considered harmful* some have uses, whilst others are symptoms of specific needs. Elm of course, being a purely functional language (by which I mean it is devoid of any side-effects), does not have global variables. It does have global *constants*. I feel like there is a missing feature in between these two. A global constant that is initialised on startup, perhaps by the program flags. 

Why would I want this? The alternative is to place such a value in the model. This is fine, but it means that anything that uses it cannot itself be a global constant, it must take in the current model. This can simply be 'awkward' at times.

Note, this feature would not get rid of common 'init' states in the model where you have to fetch data or something. This forces you to have a `Maybe a` type on your model, when actually the data is generally a `Just` value, barring a small period at startup. 

I can see why such a global initialised constant could be harmful. At some point in the future you might wish to 'update' this value, and at that point you're going to wish it was stored on the model, as you would have been forced to do had this feature never existed. Because you've been using it as a global constant you might find you have a bunch of refactoring to do. So, I think this *feature* is probably not a good idea.
