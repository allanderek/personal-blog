---
title: "Tailwind so far"
tags: programming elm
date: 2021-03-27
---

I've started using Tailwind in anger now, and whilst I cannot say I've fully completed a project I have some thoughts on it. The first thing is, I do not hate it. I can see how for a certain kind of development team, in particular single developers who are going to do the styling themselves anyway, doing the styling in whatever language you're using to generate your HTML is quite liberating. In particular I do not have quite the same fear I have of changing the styling. In large projects I find they get to the point where I start adding more and more class names to the elements so that I do not have to *modify* the current CSS but rather add more. That way I'm more confident of not breaking anything that is currently working. However, of course that's not a sustainable path forward. So I'm pretty pleased that the tailwind approach is seeing me more willing to change the current styling.

I thought I would dislike the abbreviated names, and to a certain extent that's true. I still prefer to write (whether in a separate style file, or an inline style), something like `margin-left: 1em;` rather than `ml_4`. However, I'm getting used to it pretty quickly, and I do not feel like I have to *suffer* it.

So it's working pretty well for me so far, however, I'm unsure whether it's Tailwind, or more generally generating your styles inline using the language you're using to generate your HTML, in my case Elm. I'm feeling a bit more as if Tailwind isn't actually bringing a lot to the table.  In the canonical [case for Tailwind](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/) the creator of Tailwind Adam Wathan, does address this point:
> It's easy to look at this approach and think it's just like throwing style tags on your HTML elements and adding whatever properties you need, but in my experience it's very different.

> With inline styles, there are no constraints on what values you choose.

> One tag could be font-size: 14px, another could be font-size: 13px, another could be font-size: .9em, and another could be font-size: .85rem.

Okay sure, and I do like that some styling constants, such as colours, are pre-chosen for you. Usually, if you take decisions away from me, I'm going to be pretty happy about that. That being said, I'm not sure I agree with the last part of this. Because you're in a general purpose programming language, you're free to define constants for yourself. I guess in some way I would just be re-implementing tailwind. Still though, I could give them nice names. 
So my overall conclusion so far with Tailwind, is that I'm neither detesting it, nor blown away with it. Which is a bit of a surprising result. I would have thought it was a love-it-or-hate-it, kind of thing. 
