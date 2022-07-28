---
title: "Elm open source model"
tags: elm syntax 
date: 2021-01-14
---

There was a thread on the [Elm discourse](https://discourse.elm-lang.org/) regarding the [communcation of expectations when contributing to Elm](https://discourse.elm-lang.org/t/communicating-about-elm-contributions/6729/32). It's an interesting thread albeit mostly quite negative, but it's still useful to hear people's frustrations and view points. I wanted to focus in on one particular view point that seems pretty common.

This is a direct quote from the thread:

> Thinking this a bit more, this actually sums up clearly the difference between Elm and other open source projects:

> Other open source projects:

> When issue is reported, main question is whether reported issue has merit. If it has, it will be acted upon, regardless of whether the person reporting the issue is known to the project or not.

> Elm:

> When issue is reported, main question is whether Evan has asked the person reporting the issue to contribute to Elm or not. If Evan has done so, issue will be acted upon. If Evan has not done so, issue will be considered unsolicited advice and will not be acted upon, regardless of whether the issue itself has merit or not.

I think this doesn't characterise things well, I'm pretty sure I've seen counter-examples. However, the general vibe is that whilst Elm is an open source project, the development of the main compiler and eco-system management tools is mostly a closed shop. It seems pretty opaque as to how one might go about becoming part of that closed shop. I understand that this seems in stark contrast to many other open source projects that seem to actively plead for new contributors. 

It's that very point that I wish to explore, *Elm is an outlier*, you almost might call it an experiment. I'm happy that such an experiment is being conducted, there are many many open source projects that are conducted in the more traditional open style, what if having a closed style is more productive, perhaps in the long term? How would we know? You don't have to believe that this is the case to wish to encourage the experiment. 

The key point here is that if you're un-willing to participate in such an experiment then there are many other open source projects that you can participate in. Purescript and Bucklescript both represent languages with similar target audiences and both have a more open development model. In particular Purescript is pretty similar to Elm in many ways, and the ways in which it is not are mostly due to the openness of the development model. For example, Purescript has a package management system which is much more traditional in that anyone can publish any kind of package. Elm is much more closed and so-called "kernel code" can only be written in libraries with the special status of being under the `elm` or `elm-community` namespaces. The point is, try to see Elm's development model as an experiment, if you wish to participate that's great, otherwise try another project, but please do not try to change the experiment.

Two obvious responses to this, that I'd like to address:

Firstly a reasonable response is something along the lines of *"I do wish for such an experiment, but Elm represents a major language in our production product, I wish that experiment were done in something a little less important"*.  However, to ask this question is almost to answer it, it's not much of an experiment if it is performed on a small unimportant project. I'm sorry, but if the experiment is to be done, it has to be done on something large and important.

Secondly, I imagine responses along the lines of *"I prefer Elm because {insert some reason}, otherwise I would adopt Purescript instead, but since I prefer Elm I'd rather change the way Elm is governed for the better."* That's great, but isn't it possible that you prefer Elm as a result of its hitherto closed development model? The reasons inserted above are often "eco-system is better", or "the language is simpler". But I think both are at least plausibly the result of a more closed source development model. It's possible that packages work a bit better together because most library writers are more constrained in what they can do. The language is possibly simpler because 'enhancements' from anyone who wants just a small addition to the language have not traditionally been accepted.

## Forks

Whenever such threads ignite there is usually some talk of a fork. I'd be pretty happy to see such a fork. I wouldn't join the fork, I'm happy with the way Elm is for now. However a fork would be a great hedge against the failure of this experiment.

Remember, if you're part of the Elm community you've chosen it for a reason. Presumably it fits your needs better than any of the other (many) open source languages around. Why is that? Maybe it is, at least in part, because of this closed development model. On the other hand, maybe it is because of some lucky combination of choices that have thus far been made. In the first case, I'm happy for Elm to continue as a closed development model. In the second case, **and** under the assumption that a more open development model would lead to more rapid progress, then any fork would indeed make more progress to an even more productive language/eco-system. So in that case I'd be happy for the fork to be there and presumably at some point I'd understand I'd backed the wrong horse, but presumably being able to switch to the fork would be possible.


Another benefit of a fork would be an even more convenient response to calls to open up the Elm development model. Suppose there was a fork of Elm, called Pine, then the suggestion to the frustrated would-be contributors would instead be "oh that's great that you wish to contribute, our sister project Pine is an excellent place to make those contributions". 

So in other words, I'd be happy to see a fork, because then I'd have two excellent projects with different development models to choose from.



## Conclusion

I understand a lot of the frustrations at the closed development model. I am also, personally, far from convinced that this 'closed development model experiment' will be successful. However, first of all, I really enjoy working with Elm and so I'd have to say that thus far the experiment has been pretty successful. Secondly, even if it doesn't turn out well, I'm again glad that such experiment is being performed. 
