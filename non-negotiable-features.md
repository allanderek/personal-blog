---
title: "Non-negotiable features"
tags: programming
date: 2021-03-15
---

I came across a nice [blog post](https://adam.nels.onl/blog/an-oo-languge-for-the-20s/) regarding what a modern object-oriented programming language would look like. Not now, nor never really, having been an object-oriented programmer, I'm not going to comment on how suitable the proposed language features would be. However, I thought it was an interesting exercise, many of the programming languages I fail to appreciate have made similar mistakes of ignoring things we seem to have learnt pretty conclusively about programming languages. But that's not what I wish to talk about, what I want to talk about today is the list of five non-negiotable features the author lists for any new object-oriented language (though I think he roughly means these would be non-negotiable features for any *application* programming language. Here are the five that the author lists:

* No nulls
* No unsafe casts
* Optional named arguments
* Generics
* Immutability by default

To me, one of these things is very much not like the others. The middle feature *'Optional named arguments'*. I'm trying to understand exactly why I feel that the others are quite reasonably non-negotiable but that 'Optional named arguments' are, **to me** very much negotiable. I think this mostly comes down to the fact that I can mostly achieve the effect of optional named arguments with other approaches. However, let's go through each of the other features and try to explain why they *are* reasonably non-negotiable.

Bear in mind, I'm not saying these features are necessary for any modern programming language. I'm saying that it's reasonable to have these as non-negotiable, because they mostly define your language. For example, there are arguments to be had for the case for statically/dynamically typed languages, so it is reasonable for any new programming language to be either, but it is also reasonable to say that for your particular use cases you will demand that a language be one or the other.

## No nulls

I think the author really means no null pointer runtime errors. So either no nulls, or if references can be null then you have to check them for that before using them. This essentially turns all references into a simple `Maybe` type. This basically implies two things:
1. You will have tagged union types
2. Your language is going to be statically typed.

If you have no tagged union types, then it's pretty awkward to have a value that might be something with a bunch of data, such as a 'User', or nothing, because you do not have any data about a particular User, perhaps a look up failed. It is **possible**, you could have a base 'AnyUser' class, and then both 'User' and 'Anonymous' are sub-types of that, or have 'User' a sub-type of 'Anonymous'. I feel that such approaches are work-arounds to not having the apparatus (tagged union types) to really say what you mean eg. *'there is no user data'*.  

If you have a dynamically typed language, then you have the possibility of getting runtime errors, that's just part of the tradeoff. For example, you might claim that Python doesn't have `null`, but it does have `None` and any value you try to use as something that is not `None` may in fact be `None` and when that happens you will get a runtime exception.


The case for this feature is pretty clear. You can guarantee a bunch of errors will never happen. 


## No unsafe casts

This is basically a generalisation of the above point. If you have no nulls in your language you cannot do an unsafe cast (whether implicit or explicit) from a 'possibly-null' value to a 'definitely-not-null' value. So this feature basically generalises that and states I do not want any runtime errors from attempting to treat a value of one type as a value of a different (incompatible) type.

This feature implies static typing, since in a dynamically typed language you are essentially **always** doing unsafe casts. This also either implies generics or implies that you cannot write generic code. Suppose you write a method to reverse a list. If you have no generics then in order to use this method on different kinds of lists, ie. in order for this method to be generic, then it must operate over the most generic type. Typically in an object-oriented language this would be something like `Object`. So you can reverse a list of strings by downcasting it to `Object`. That is safe because `String` would presumably be a sub-type of `Object`. However, once you have your reversed list, it is necessarily a list of `Object` values. In order to treat each element of that list as a `String` you would need to perform an unsafe cast. The way around this is to have generics so that you can write a reverse method that takes in a list of *'somethings'* and returns a list of *'somethings'*.

## Generics

This is mostly the same as the above 'No unsafe casts', for the reasons identified above. I think this is **why** generics is so non-negotiable, it **enables** *no unsafe casts*, which in turn subsumes *no null*.

## Immutability by default

So I confess this is a little more difficult to see why it is non-negotiable. It's certainly a nice feature. I think there is growing evidence that immutability does indeed prevent some kinds of bugs. As I've [argued before](/posts/2021-01-23-immutabilit-bugs), immutability also makes a certain kind of bug **more** likely. However I think immutability is generally accepted now to be a net-win for correctness. Nonetheless I think the evidence is not yet available for whether or not 'immutability by default' is a reasonable compromise between allowing mutability and completely forbidding it. Though at least at this point, if you're set on allow mutability, then you have to have either mutable or immutable as the default, and at this point it seems pretty safe to bet on immutable as the better default.

## Odd one out - optional named arguments

So the previous few features I feel have a pretty large bearing on the kind of programming language you are building. Each of these features will have pretty profound effects on the types of programs that can or should be written in that language. It is clearly a language aiming for correctness. By contrast *optional named arguments* just seems like a small convenience. I'd put it firmly in the category of personal preference. Personal preference are those features that you cannot build a particularly compelling argument either for or against, you just either like it or you don't. Personally, I don't have much against optional named arguments. I prefer named things to anonymous things ([I don't think Elm](/posts/2021-01-24-lambdas) or [Python](/posts/2021-01-30-lambdas-again) really needs anonymous functions). Nonetheless I think for optional arguments you can just use a `Maybe` type. This would solve [the top Python gotcha](https://docs.python-guide.org/writing/gotchas/).




