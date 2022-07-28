---
title: "Paul Grahams's Programming Language Questions"
tags: programming elm 
date: 2021-03-09
---

I'm not sure if it is intentional or something to do with the RSS configuration on his site but Paul Graham's RSS feed is going through a redux of posts from 2001. Some of these are interesting to read with the 20 years worth of perspective. Back in 2001 [he thought that Java didn't smell great](http://www.paulgraham.com/javacover.html), a point he made without trying the language. I think for the most part this post has stood up well to time other than the frequent use of Perl as a counter-balance. 

Today there is a [post regarding programming language ideas](http://www.paulgraham.com/langdes.html), I thought I would go through each of the head-lined points and put Elm into that context. 


### Programming Languages Are for People.

I think Elm does pretty well here, he doesn't specifically talk about error messages, but clearly Elm has broken new ground with its compiler error messages. He does say (my emphasis):

> And when I say languages have to be designed to suit human weaknesses, I don't mean that languages have to be designed for bad programmers. **In fact I think you ought to design for the best programmers**, but even the best programmers have limitations.

Elm is pretty hardcore designed for beginners, beginners to Elm rather than beginner programmers, although implied in that is beginners to functional programming. All of this is not to equate beginners with *'bad'*. Nevertheless, I do feel that at times Elm's design neglects the experts. I have [suggested](/posts/2021-01-18-terser-error-message/) a `--terse` flag to give shorter error messages aimed at veteran Elm programmers, and it's one of a few points in which veteran Elm programmers could be better served.

### Design for Yourself and Your Friends.

I think Elm does pretty well here too. I think I understand the point he is making, I think Go has fallen into the trap that he speaks of. I would, perhaps erroneously, like to think of myself as a decent enough programmer, and I believe Go was explicitly designed to be 'easy'. So perhaps that's why I cannot [grasp why Go is popular](/posts/2021-02-22-go-not-for-me/). Anyway, as far as I can tell Elm does okay here.

### Give the Programmer as Much Control as Possible.

I think Elm is at pretty large odds with this design principle. Often Elm stops you doing things that its designers believe are not good for you, most notably in that published libraries cannot make use of so-called *'kernel-code'*. I personally think Elm's approach has proven pretty fruitful, but having said that it certainly generates a lot of complaints. This is something you either agree with or not, I can see how it might *feel* to be treated as less than an equal, but don't forget the point the author has made in the first section: *"but even the best programmers have limitations"*.

### Aim for Brevity

I think I mostly disagree with this. I think the general principle of *"Aim for brevity"* is just wrong, you should neither aim for brevity nor length. Brevity where appropriate is fine. I think Elm gets this correct, it favours being explicit over the implicit.

Recently Rupert Smith said the following the on Elm discourse which I thought very elegantly articulated his thoughts on boiler-plate code in Elm which I agree with:

> I also think that in many programming languages boilerplate is understood as a bad thing that must be eliminated. So in Java or Angular we get lots of @Annotations, each of which is a short cut to some piece of boilerplate. But then hardly anyone has any idea what the program is actually doing and tracing bugs in the annotations is a nightmare. In Elm we do not regard boilerplate as a bad thing and prefer to be explicit over being concise. It took a good while of using Elm for me to appreciate this.

However, whilst I disagree with the general point of always aiming for brevity I think he is spot on here when he states:

> And it's not only programs that should be short. The manual should be thin as well. A good part of manuals is taken up with clarifications and reservations and warnings and special cases. If you force yourself to shorten the manual, in the best case you do it by fixing the things in the language that required so much explanation.

Here here, and I think Elm gets the very right. It has a famously small grammar among other things which are kept small.

### Admit What Hacking Is

This is a little more difficult to get at the real point that is being made, but it is something regarding the difficult part of language design is more figuring out how different features fit together into a coherent whole, rather than having some unique feature, particularly one that embodies some theoretical/academic breakthrough. I think Elm does this in spades. In fact, I think this is one of the things that I like most about Elm. I realise that the whole idea of a shadow DOM with the Elm Architecture was fairly experimental when Elm was first conceived. However, the Elm architecture aside, the actual **language** is mostly a collection of features from other functional languages combined in as pragmatic a way as possible. Elm looks a lot like Haskell, but strips away many of the unnecessary features, and greatly improves on record types. It also has some of the same operators but they are designed to work together well so `<<` and `<|` indicate results being passed backwards and `>>` and `|>` indicate results being passed forwards. This is greatly superior to Haskell's infix operators for the same thing in which `.` is equivalent to `<<` and `$` is equivalent to `<|`.  Elm has observed the fact that Hindley-Milner style polymorphic typing is major boon to the programmer, but has agumented this by focusing on achieving better error messages to the user.

I contrast this with Go which I believe has focussed on a couple of *'fancy'* features, such as *go routines* and largely ignored most of what programming languages have taught us over the preceding two decades (90s, and 2000s before Go was founded). Just why-oh-why would such a language not be designed with generics from the start?


## Conclusion

I think this now twenty year old post stands up to the ravages of time pretty well. I don't agree with everything in it, but they are nonetheless valid points. I also think Elm has done pretty well (presumably accidentally) in adhering to many of these principles. It is clearly a language that agrees with me, as the points in this post that I take issue with are the same points that Elm has not stuck by. There is more at the [original post](http://www.paulgraham.com/langdes.html), including *Open problems* and *Little known secrets* that I may cover in other posts.
