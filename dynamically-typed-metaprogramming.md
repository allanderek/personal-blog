---
title: "Dynamically typed languages and meta-programming"
tags: programming
date: 2021-01-31
---

I'm going to go a little nostalgic and explain my journey through thoughts on statically/dynamically typed languages. I'll arrive at the conclusion that (some) dynamically typed languages have mostly nailed meta-programming. Whilst it still seems something of a todo for statically typed languages.


## Dynamically/Statically typed languages and productivity

When I first started programming, for a while, I was a card-carrying member of the strongly, statically-typed languages fan club. I didn't really understand why anyone would ever wish to remove type type-checker which I saw as a major help to the programmer. In other words, I couldn't really understand why anyone would wish to write code that **did not** pass a static type-checker.

I then tried Python. I found it very productive. I was still niggled by the thought that perhaps Python (or dynamically typed languages in general) were productive in the *short* term, but less so in the long-term. In other words, you could quickly write code to satisfy the requirements of the day, but you ended up with less maintainable code. I could see a world in which such a language became very popular, and because it was popular it has a lot of support in terms of IDEs, and, most importantly, libraries. That then reinforces the idea that the **language** is more productive, when actually it's the eco-system that is providing much of the productivity boost, and it's a productivity boost in the short-term only. Still, I was pretty happy programming in Python, even if it irked me a little that I couldn't quite explain the effectiveness that I felt (real or imaginary).

Certainly, a part of my thinking at the time went something like this: you should write tests for all of your code, you should have 100% test coverage. Once you have that, that's better than a type-checker, and the type-checker doesn't buy you much on top of that. It might even be that the lack of a static type-checker somewhat enforced you to write decent tests for your code. There's even some psychology-related results that might support this hypothesis. [Here's a good blog post](https://www.johndcook.com/blog/2010/06/09/dynamic-typing-and-risk-homeostasis/) explaning why, the first sentence is pretty much most of the explanation:

> When we make one part of our lives safer, we tend to take more chances somewhere else. Psychologists call this tendency [risk homeostasis](https://en.wikipedia.org/wiki/Risk_compensation#Risk_homeostasis).

Okay so that might explain a little why a dynamically typed language could be more productive even in the long run. But still I found it a rather unsatisfying explanation. This sounded like something that would have a small effect over a longer time-frame. Why were dynamically typed languages taking over the world of programming? I could see why with a full test suite, the type-checker might not provide **much** benefit, still, I found it hard to believe that benefit was zero, and I still couldn't understand why a type-checker might be **detrimental** to productivity, at least not appreciatively.

## Meta-programming

So I couldn't explain what made certain dynamically typed languages more productive, at least under certain conditions, but it certainly seemed like they were. Then I started using Python to make web backends. If you've done this before, you'll realise that they come with pretty decent support for accessing databases. At least as far as ORMs go. Not only that, but you typically get a database admin interface, **for free**.

This is pretty incredible, you write your datatype models to **access** the database, and the framework is able to automatically generate a database admin interface. Imagine doing this in a statically-typed language. Suppose you have a page to update a row in a database. In a dynamically typed language you can store this row as a dictionary mapping column names to values. In a statically-typed language all the values would need to be of the same type. Hence they would be of some kind of union type. It's tricky to explain how awkward this gets, when you're writing the code to update each of these fields, but it does. What do you do when you attempt to update a 'boolean' field with a 'date'? In the dynamically typed language you just do not consider this possibility because it's impossible, and you write tests to make sure it isn't. In addition this code is so generic that you iron out the bugs for everyone. But in a statically typed language you're forced to consider the possiblity, and do **something**. At best what you find is that you end up writing an interpreter for a mini-dynamically typed language. See also [Greenspun's tenth rule](https://en.wikipedia.org/wiki/Greenspun%27s_tenth_rule).

## Conclusion

My tenatative conclusion then is that meta-programming is just easier in a dynamically typed language, because it's basically just normal programming. You already by necessity have access to reflection (eg. determining the type of a given value), something that you don't have in statically typed languages because you're supposed to know the type of a value. 

To be clear, there have been attempts to afford statically typed languages with meta-programming capabilities. The two I am most aware of are [Fresh O'Caml](https://www.cl.cam.ac.uk/~amp12/fresh-ocaml/) and [Template Haskell](https://wiki.haskell.org/Template_Haskell). However, for some reason I don't know of any meta-programming extension/framework for a statically-typed language that has become popular/successful/mainstream. I'm not at all clear why. I plan to explore meta-programming for statically-typed languages in some future blog posts, but of course [best laid plans](https://en.wikipedia.org/wiki/Of_Mice_and_Men) are apt to [gang aft agley](https://en.wikipedia.org/wiki/To_a_Mouse) and all that.
