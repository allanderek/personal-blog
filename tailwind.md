---
title: "Tailwind CSS"
tags: programming
date: 2021-03-06
---

A [Tailwind CSS package](https://package.elm-lang.org/packages/matheus23/elm-default-tailwind-modules/latest/) was announced on the [Elm discourse](https://discourse.elm-lang.org/) which lead me to checking out [Tailwind](https://tailwindcss.com/) which in turn led to [this post](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/) by the author of Tailwin. In it he makes an excellent point which I'm going to try to distill into a smaller form, though you're well advised to read the [entire original article](https://adamwathan.me/css-utility-classes-and-separation-of-concerns/).
The author's main point is that try as you might to separate the semantic concern of the HTML from the display concerns of CSS, one or the other will depend upon the other. In particular when using semantic classes in your markup, your CSS becomes a mirror of your markup structure. That means that if you change the HTML you likely have to change your CSS. Note, when I say CSS here, I mean, however you write your styles, using LESS, or Sass, or, if you prefer, vanilla CSS.

The author then details several ways in which you might try to decouple your styles, which you can do to some extent, but it comes at the cost of re-usuable CSS components.

This leads to the author's main point, that separation of concerns is not the important way to think about styling and markup, instead, the important distinction is *'dependency direction'*. Here he defines two methods to writing styles and markup:

#### Css that depends on HTML

> *"The HTML is independent; it doesn't care how you make it look, it just exposes hooks like .author-bio that the HTML controls."*

#### HTML that depends on CSS

> *"The CSS is independent; it doesn't care what content it's being applied to, it just exposes a set of building blocks that you can apply to your markup."*

Neither of these two styles is inherently wrong, it depends upon your projects and what you wish to maximise for. If you wish to maximise for reusable styles, then you should opt for the second approach. 


## Elm

In Elm we have several possibilities for styling. Some of these include doing the styling in Elm, either using elm-ui, or elm-css. There are many advantages and disadvantages to this. The one advantage I'd like to highlight right now, is that all of your *abstraction* is done in Elm, a general purpose programming language well designed for abstracting. This fits well with the second approach defined above, because at some point you will need to compose (ie. combine) some of these resuable styles. Doing so in Elm can be pleasure.

