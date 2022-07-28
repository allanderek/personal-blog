---
title: "Is Elm faking work"
tags: programming elm
date: 2021-03-10
---

It's always worth examining the things you love for the reasons why. Could something you think is improving your productivity actually be harming it? Perhaps in a way that makes it feel like you're being productive.

Continuing the look at some of Paul Graham's older posts, [this one regarding object-oriented programming languages](http://www.paulgraham.com/noop.html) popped up on my feed. I think it stands up pretty well over the previous two decades. I wouldn't say object-oriented programming has been debunked, but it has certainly lost some of its hype. I thought about Elm when reading the following third point:

> Object-oriented programming generates a lot of what looks like work. Back in the days of fanfold, there was a type of programmer who would only put five or ten lines of code on a page, preceded by twenty lines of elaborately formatted comments. Object-oriented programming is like crack for these people: it lets you incorporate all this scaffolding right into your source code. Something that a Lisp hacker might handle by pushing a symbol onto a list becomes a whole file of classes and methods. So it is a good tool if you want to convince yourself, or someone else, that you are doing a lot of work.


This is the third item in a list of five, which are reasons the author thinks programmers like object-oriented languages, he says that 3 and a half of them are bad.

At some point I was somewhat forced into programming in Java and I never really got on very well with it, although I certainly didn't feel like as if I was fighting the language. I recall a vague sense of feeling as if I was filling in forms. I do get a similar feeling with Elm on occasion. When I create a new input and I have to also create a new message, and then I have to fill in the case in the `update` function to handle that new message. It can feel *'fiddly'* and *'form-fillingy'*.

So the question is, is it possible that the Elm architecture is rigid enough that you feel like you're doing a lot of work, but not actually getting much functionality produced? I cannot really explain why I don't think this is the case for Elm. I would definitely like to return to this question when I have a better answer for it.
