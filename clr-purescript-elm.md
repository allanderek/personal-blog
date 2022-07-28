---
title: "CLR, Purescript, and Elm"
tags: elm  
date: 2021-01-03
---

I'm going to talk about the common-language-runtime, and use this to contrast Purescript and Elm. Although I have in mind the .Net common-language-runtime, I'm more talking about the general concept. Specifically, it's not just the common platform used as compilation target, but the idea of interoperability, in particular libraries that can be consumed by different languages so long as they adhere to the common-language runtime. Similarly whilst I'm contrasting Elm to Purescript, I'm more contrasting narrow versus broad scopes.


## Functional languages and popularity

I say functional languages, but more generally *niche* languages have something of a bootstrap problem. Languages are easier and more productive to work in, if there exist a rich set of accompanying tools and libraries. Because such languages are more popular more libraries and tools are likely to exist for such languages. A niche language doesn't have the associated libraries and tools because there aren't enough developers interested in building such tools because by definition the language is niche and unpopular. I suspect that the perceived productivity of some popular languages is at least partly due to the fact that more extensive, mature, and maintained libraries exist for such languages. This is especially true of languages with commercial backing, meaning that developers can be paid to write the more boring libraries.


## The common language runtime and piggy-backing

The common language runtime, was seen as a potential solution to at least part of this problem. The idea, roughly speaking, of the common language runtime, was to say that many languages have different features, but share similar underlying semantics. In addition many libraries essentially solve the same underlying problem. So, it should be possible to write some libraries *once* and, if you compile multiple languages to the same *runtime* then they should be able to inter-operate, without any *glue code*. 

## Problems with piggy backing

One issue with this idea is that many libraries, if written in separate languages, would have different interfaces.
So whilst it is handy to allow say, a functional language like Haskell, access to some GUI library on the .NET runtime, it means you end up writing your Haskell application in a pretty non-functional manner. So in this way, the library *infects* the application code.

Furthermore the writer of the library can make fewer assumptions about the context in which their library will be used.

Unfortunately this can mean you end up with multiple libraries that perform the same function, but in a different way. So for example, you might have a red-black tree implementation with a very object-oriented interface, and a red-black tree implementation with a very functional interface. This means you tend to have more libraries that do very similar things, and choosing between them can become challenging. Maybe there are multiple JWT libraries with different capabilities but also differently styled interfaces. Suppose you're writing a Haskell application, that requires a JWT implementation that respects expiration on the token. Do you use the JWT library that exposes a very object-oriented interface but implements token expiration for you, or do you use the JWT library that exposes a functional interface, but you'll have to build the token expiration functionality on top of it yourself? Also the second library isn't as well tested or popular, but you cannot tell if that's because Haskell is less popular or the library is simply not as good.

## Purescript and Tea

Purescript and Elm are two languages with a fair amount in common. Both are *pure* functional languages, both are primarily designed to compile to Javascript, both are primarily intended for use in writing code to be run via Javascript in the browser (e.g for making the front-end of web applications). However, Purescript is far broader in scope, Purescript can be used to write the backend server code, even restricting yourself to writing front-end web code, Purescript does not have a built-in artchitecture. Elm has a built-in architecture known as TEA (The Elm Architecture), Purescript allows you to build that architecture on top of the standard library, but therefore allows more flexibility because you are not **restricted** to building your application in that way.

In Elm it is possible to fight against the built-in architecture and write your application in a different way, but you're likely going to have a pretty bad time if you do that. For this reason, basically all Elm applications are written using TEA.

So Elm appears more restricted, however it has a major benefit. All libraries written for Elm, assume that the consumer is writing their application using TEA. This reduces (but by no means eliminates) the proliferation of libraries that solve the same problem but makes different assumptions about the use-case. For example, you might write a library for a date-picker, in Elm it's far less likely that this work will be duplicated because the original library doesn't work for some consumer, because it assumes a particular kind of architecture. 


## Conclusion

To be very clear, I'm **not** suggesting that the *"Elm way"* is better. Only that it has this advantage. The complementary disadvantage should be pretty clear, it means Elm is useful in fewer circumstances. If you limit the scope of some language, or more generally tool, then you also broaden the set of assumptions that accompanying tools are allowed to make. Here is another example, if your language is a **pure** functional language, then an optimising compiler can make more assumptions about which transformations are valid.

The takeaway, if there is one for this post, is that whenever you feel *'constrained'* by a language or tool, consider the possibility that those very constraints come with a significant benefit.
