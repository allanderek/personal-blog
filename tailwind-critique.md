---
title: "Response to a Tailwind Critique"
tags: programming elm
date: 2021-03-17
---

Sometimes it's possible to make a kind of judgement of a technology without trying it out, and that's useful. In this sense you're judging a book by its cover, see [Paul Graham's pre-critique of Java](www.paulgraham.com/javacover.html). Because of the recent [release of Tailwind modules for Elm](https://discourse.elm-lang.org/t/elm-tailwind-modules-first-release/6899) I've been trying to judge the cover of Tailwind before I jump in and start using it. One way of judging a technology without trying it, is to look at critiques of that technology. Do those comments hit home for you? A prominent [critique of Tailwind](https://www.aleksandrhovhannisyan.com/blog/why-i-dont-like-tailwind-css/) came up on my feed. I think the author of this critique just doesn't share similar problems to the ones I typically face, and, I **think** are typically faced by Elm developers. So I'm going to go through most of the points in the critique article and roughly state why I don't think they really apply for Elm developers.

The author makes 7 main points against the use of tailwind. I believe he is mostly talking from the perspective of writing *mostly* straight-up HTML. It might be HTML that is in a template and so is technically generated HTML, but the template feels very much like writing the full HTML out. Because of this, I think most of his points are not relevant to an Elm developer.

## Tailwind Makes Your Code Difficult to Read

There are at least three points made within this heading, so I will try to address all three here. The first is that the class names are *'awkward abbreviation'*. Okay that's difficult to evaluate, but isn't really a point against the philosophy of Tailwind, rather the implementation. 

The second point here is:

> Another reason why Tailwind is so hard to read is because it requires you to pan your eyes horizontally rather than vertically. 

To which the example given is:

```html
<div
  class="w-16 h-16 rounded text-white bg-black py-1 px-2 m-1 text-sm md:w-32 md:h-32 md:rounded-md md:text-base lg:w-48 lg:h-48 lg:rounded-lg lg:text-lg"
>
  Yikes.
</div>

```

Well okay, but in Elm we would write this as:


```elm
let
    classes =
        [ "w-16"
        , "h-16"
        , "rounded"
        ...
        ]
            |> List.map Attributes.class
in
Html.div attributes []
```

Or with type-checking using the aformentioned tailwind-elm package. The point is though the classes would almost certainly be written vertically. The author claims that you cannot write them vertically in HTML because of ESList/Prettier. I do not know enough about either, but fair enough.

A **third** point in the first section is regarding the separating of 'media' styles, for small screens, medium screens and large screens. However if you have the full power of a general programming language, you can of course split up the list anyway you like:


```elm
let
    allScreens =
        [ .. ]
    smallScreen =
        [ .. ]
    midScreens =
        [ .. ]
    largeScreens =
        [ .. ]
    classes =
        [ allScreens
        , smallScreen
        , midScreens
        , largeScreens
        ]
            |> List.concat
            |> List.map Attributes.class
in
Html.div attributes []

```

## Tailwind Is Vendor Lock-in

> If you use Tailwind, you’re stuck with it, unless you can tolerate converting all of that CSS to semantic CSS by hand.

I think this point can be conceded. I'd say that's mostly true regardless of how you choose to do your styling. Sure if you use classes in your HTML and target those in your CSS, then you can get rid of all of your CSS and start again with a different CSS framework, but only one that expects to work in a similar way. So basically whatever your *philosophy* for styling is, it's difficult to change that mid-project.

## Tailwind Is Bloated

This might be a reasonable point. But it doesn't really apply to use Tailwind with Elm, at least not using [Elm tailwind modules](https://discourse.elm-lang.org/t/elm-tailwind-modules-first-release/6899) since the Elm compiler will purge out anything that is unused (if I understand correctly). 

## Tailwind Is an Unnecessary Abstraction

> With Tailwind, if you want to group related styles together under a reusable class name, you need to use @apply directives, like in this example taken from the docs:

Again with a full general programming language behind you this mostly goes away. Whereas in HTML you might write:


```html
<button class="btn-indigo">
  Click me
</button>

<style>
  .btn-indigo {
    @apply py-2 px-4 bg-indigo-500 text-white font-semibold rounded-lg shadow-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-400 focus:ring-opacity-75;
  }
</style>

```

In Elm you can instead write:


```elm
buttonIndigoClasses = [ py2, px4 ... ]

...
    let
        onClick =
            Events.onClick Msg....
    in
    Html.button
        ( onClick :: buttonIndigoClasses)
        [ Html.text "Click me" ]

```

## Semantics Is Important. Tailwind Forgoes It.

> Supporters of the framework usually respond by reminding you that you’re probably working with a component-based framework like React anyway, and so you’re unlikely to be looking at a giant sea of markup in a single file. Fair point.

So I think the author is here conceding that, yeah if you're not directly writing your HTML his point doesn't really stand. I think though that same concession could apply to most of the points made. 

## Tailwind and Dev Tools Don’t Play Nicely

Again this is a pretty fair point, but if you're doing your styling in **Elm** as opposed to HTML they are broken for that use case anyway. At least broken in the way that is described here. However, yes it is a positive for doing your your styling in CSS that can be directly edited by the dev-tools in the browser. That's one of the *general* advantages of doing your styling in CSS rather than in Elm/whatever language you're using to generate your HTML.

##  Tailwind Is Still Missing Some Key Features

Okay this is not a point against the Tailwind philosophy but again the implementation.  It seems a fair criticism. 

## Conclusion

Ultimately, I think this is a pretty simple tale of whether you wish to do your styling in CSS or in the language you're using to generate the HTML. There are some advantages and disadvantages to both. But most of the disadvantages of doing your styling in HTML spoken of in the linked to article, are just that, disadvanages of doing the styling in HTML directly, rather than in some full general purpose language that is used to generate the HTML.

Nonetheless the [critique article](https://www.aleksandrhovhannisyan.com/blog/why-i-dont-like-tailwind-css/) is worth reading.
