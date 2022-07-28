---
title: "Could meta-programming be harmful?"
tags: programming
date: 2021-02-01
---

Could meta-programming be considered harmful? Probably not, but like all features of programming languages it will not automatically improve your code. Because I intend to talk in some future posts a little about meta-programming in statically typed languages I want to first lay down some caution. In other words, I do not wish for it to be taken as granted that meta-programming is a solid win, a feature that needs to be added to statically typed languages. 

There have been various attempts to reduce the core of a language such that other forms used whilst actually programming are just forms defined by the standard library. To take a straightforward example of this kind of 'core reduction' think of arithmetic binary operators. One approach is to define the `+` binary operator as part of your language. Another is to allow the generic definitions of binary operators and then define `+` in the standard libary, typically as equal to some 'addition' function/method. 

At first this seems great. You can define your own binary operators, and you can even re-define the `+` binary operator. For example, if you're working on a Matrix module, you could define operators that work over Matrices. Depending on your type system there are various outcomes you might aim for. One might be that `+` just becomes usable over matrices in addition to the usual operands of integers, floats, and perhaps strings and lists. Another is that you define something like `+#` to be matrix addition and `*#` to be matrix multiplication.

The downside, is that people found that this quickly lead to what might be called *'write only'* programs, in that they were somewhat difficult to understand for anyone but the person that wrote them. That's because in order to understand a bit of code you have to **first** understand what all the newly defined binary operators are. If these are definable by *anyone* this can be non-trivial under taking. This problem is of course much worse if you allow people to define other forms within the language, say new keywords, or other new syntax.


## Dynamically typed languages

As noted yesterday one great advantage of dynamically typed languages is that meta-programming is just normal programming. It is mostly pretty painless to do in a dynamically typed language, provided it gives you access, for example provided you are able to say define new classes programmatically, as you can in Python.

This means that you can get libraries such as [attrs](https://www.attrs.org/en/stable/examples.html) which essentially change the semantics of the language. The `attrs` library is a nicer way of initialising objects which are mostly record types perhaps with some related methods. It is very nice to work with. This is incredibly powerful, when you think about it, *'attrs'* makes working with record-like types quite pleasant, using just the classes that Python already gives you. Other libraries could in theory introduce different styles for different kinds of types. All are just Python classes (and their instance objects) under the hood, but *feel* like you're programming with something different.

However, the danger in defining such libraries is it that introduces yet another  barrier for someone reading your code. Even *'attrs'* means that there are now two ways define a very simple class in Python. Suppose you want a class to hold a 2D *'point'*, now you can either define it in vanilla Python, or use *'attrs'*. This choice, is both a burden on the developer writing the code, and something that someone reading (or trying to debug) your code is going to have to first understand.

If we have learnt anything from programming in its first several decades it is that we should optimise the experience for reading, understanding, debugging, and maintaining code, not writing it in the first place. There is a danger with meta-programming that you optimise for writing code.


## Conclusion

Nonetheless meta-programming done well is surely a major boon for developers. You just have to remember that it carries the risk of making your *actual* programming code harder to read. At it's heart, or best, meta-programming is about being able to make fewer changes, not being able to use fewer key-strokes.
