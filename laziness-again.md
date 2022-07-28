---
title: "Laziness again"
tags: programming elm 
date: 2021-02-10
---

I [wrote recently about laziness](/posts/2021-02-07-laziness), I was trying to convey a sense of how useful laziness can be. I pointed out that it does have some disadvantages. I have been reading Michael Snoyman's series of [posts on the bad parts of Haskell](https://www.snoyman.com/blog/2020/10/haskell-bad-parts-1/) ([part 2](https://www.snoyman.com/blog/2020/11/haskell-bad-parts-2/), and [part 3](https://www.snoyman.com/blog/2020/12/haskell-bad-parts-3/)). It's part 1, that I'm interested in. He talks about the `sum` and `product` functions over lists, here is what he says:

> The sum and product functions are implemented in terms of foldr. Well, actually foldMap, but list's foldMap is implemented in terms of foldr, and lists are the only data structure that exist in Haskell. "Oh, but foldr is the good function, right?" Only if you're folding a function which is lazy in its second argument. + and * are both strict in both of their arguments.

With a good explanation afterwards of what it means by *'strict in both arguments'*:

> If you're not aware of that terminology: "strict in both arguments" means "in order to evaluate the result of this function/operator, I need to evaluate both of its arguments." I can't evaluate x + y without knowing what x and y are. On the other hand, : (list cons) is lazy in its second argument. Evaluating x : y doesn't require evaluating y (or, for that matter, x). (For more information, see [all about strictness](https://www.fpcomplete.com/haskell/tutorial/all-about-strictness/).)


So my point here is that this kind of discussion you just never hear regarding strict languages, because **all** arguments are always strict. Now you might point out that maybe we **should** more often hear about arguments which are strict/lazy in eager languages. As I pointed out, everything always being strict, means that either you end up needlessly evaluating things that you do not need to, **or** you are complicating your code by essentially implementing the laziness yourself. 

Perhaps if a strict language gets a laziness-analyser (analogous to the strictness-analyser in lazy languages such as Haskell) that can (probably only locally) detect some laziness and re-arrange your program to take advantage of it, then perhaps these kinds of discussions **would** occur.

Again, these discussions are obviously not to say that strictness is better than laziness, or laziness is better than strictness, but it depends. I feel that strict evaluation is a little more intuitive and probably appropriate for a language such as Elm that is clearly aimed at being beginner friendly.


