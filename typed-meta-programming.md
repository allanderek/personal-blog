---
title: "Typed meta-programming"
tags: Elm maintenance programming
date: 2021-02-06
---

This post is some vague, not-well-thought-out rambling on meta-programming in statically typed languages.

As I have said dynamically typed languages tend to have meta-programming already baked in. This is because changing the program itself, doesn't need to be re-type checked. However, the whole point of a statically typed language is that the program is type-checked before it is run. So you cannot then change the program at run-time because in that case the new program would not be typed. In theory of course you could allow this, but you would have to do one of three things:
1. Separate the actual running of code, from the changing of the program, so that you can then type-check the resulting program before continuing.
2. Give up on type-safety of dynamically changed code.
3. Type the meta-programming code in such a way that it can only produce type-safe code.

The third one here would be great, but it is either impossible or results in a pretty limited meta-programming capability. I do not know that the second option has been tried, it does seem a little pointless, as in, if you're going to do that, why not just go for a full dynamically typed language. Which leaves the first option, which is both challenging and potentially expensive at run-time since you're going to have to run the type-checker.

However, the meta-programming does not have to take place at run-time. I [wrote about](/posts/2021-02-02-type-classes-are-meta-programming) type-classes being a limited-form of meta-programming, but more specifically they are limited form of **compile-time** meta-programming. There is no reason you cannot provide your own compile-time meta-programming facilities. You can simply write some code which takes as input a program in your language of choice and outputs another program, after some transformation, in the same language. For Elm we already have some tools that do this. For example [elm-review](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/) can automatically fix some of the problems it detects in your code. For elm-review it is generally intended that the code-fixes keep the same behaviour but just fix some style/maintenance issue. However, you could use the same structure to simply create additional code, or change existing code. Here's a simple example. A simple enum in Elm is just a custom variant type in which each constructor has no parameters. When you have such a simple enum you often wish to enumerate all the variants, perhaps to display them all, or allow selection of one via an `Html.select` element. This is best shown with an example, suppose you have a type definining the *'roles'* a user may have in your system:

```elm
type Role
    = User
    | Admin
    | SuperAdmin
```

Now suppose you wish to have a *'create/edit user'* form, for the `SuperAdmin` user. This contains a drop-down selector for the role the user has, so for that, you ultimately want the full list of possible roles:


```elm
allRoles : List Role
allRoles =
    [ User
    , Admin
    , SuperAdmin
    ]
```

Now, what happens when you update the `Role` type? Suppose you add a new kind of role, an `AdminLight` role, that has some admin power but not all of them. A problem is that you may forget to update the `allRoles` value. One option is to write an elm-review rule that checks this. Another is to write code that actually generates the `allRoles` value, from the type definition. 


This is a path I hope to explore a little. There are many questions, such as what language should you use to write the meta-program in? It doesn't have to be your target language. It could be dynamically typed, because the code that you produce will be type-checked by the statically typed language's type-checker before being run anyway. Perhaps the most challenging part is deciding how to instruct the meta-program to do the transformation.
