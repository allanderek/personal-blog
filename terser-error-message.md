---
title: "Terser error messages"
tags: elm 
date: 2021-01-18
---

I was reading a [post](https://tilde.team/~kiedtl/blog/chatterbox/) found on Hackernews. Its main point is that shorter error messages are superior because the main problem is not obscured by other, mostly irrelevant details. I think it's far from a universal truth, but some of their examples were pretty compelling. It got me thinking about Elm's error messages.

[Elm's compiler error messages](https://elm-lang.org/news/compiler-errors-for-humans) are famously great. However, they are not typically short. This is a major boon for beginners. The compiler often explains in detail why it was not able to type check your entire program and possible fixes. So for beginners this is a major boon. For more experienced users however there is a lot of information being printed that is not helpful, mostly because we've seen it before. 

Here is an example ([taken from this blog post](https://calebmer.com/2019/07/01/writing-good-compiler-error-messages.html)):

```elm
I am inferring a weird self-referential type for x:

11| f x = x x
      ^
Here is my best effort at writing down the type. You will see ∞ for parts of
the type that repeat something already printed out infinitely.

    ∞ -> a

Staring at this type is usually not so helpful, so I recommend reading the
hints at <https://elm-lang.org/0.19.0/infinite-type> to get unstuck!
```

This a *fantastic* error for someone seeing this for the first time, or who is generally new to compilers which perform type inference. But for a seasoned developer I think:

```elm
I am inferring a weird self-referential type for x:

11| f x = x x
```
Is enough. Even if the expression is way more complicated, as soon as I see the words "self-referential" I basically know what the problem is, or at least I understand enough the remaining stuff about infinite types is not going to help me. 


So I *would* propose a `--terse` flag to the compiler, meaning "show me shorter error messages". I would probably have that in my build script, and then even if I come across an error message in which I *could* be helped out, I can easily just re-run the compiler without the `--terse` flag to see the full explanation (in particular the url).

I say **would** propsose this, but I don't actually think this should be done, I just don't think it has enough bang for the buck of the involved effort. If Elm was a commercially supported language with a core team of 50 engineers, then yes I would ask for this. As it is, the effort of separating out necessary information that is always shown (ie. what exactly is the problem), from the unecessary information ("I figure out types from left to right ..."), and then **also** maintaining that distinction is just too much work for a small team.


