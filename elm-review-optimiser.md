---
title: "Elm-review as an optimiser"
tags: programming elm
date: 2021-03-16
---

I've written before about the prospect of [compile-time laziness](https://clouddev.pakk.io:4014/posts/2021-01-28-safe-dead-code-removal) and using [elm-review](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/) to act as an elm-to-elm compiler pass.

Obviously one way to implement compile-time laziness would be to fork the compiler. As I said previously the use of elm-review to do this instead is something of a low-risk route into this. If this works out particularly well it could be translated fairly easily into an internal compiler pass. An internal compiler pass would have the benefit that the output of the pass might not necessarily have to be valid Elm. It might also be faster because it would not involve the expensive operations of unparsing and re-parsing the Elm code. 

There is a further problem in that you probably wish to run the compiler's type-checker over the original code, because you probably want to fix the original code, not the optimised version. So this means running the compiler over the unoptimised code and then once again to compile the optimised code.

However, do note, that you would only have to run the optimisation phase when deploying, not whilst developing. In addition, the fact that the code is re-checked by the compiler gives us a bit more guarantee that the optimiser is not introducing a bug.

Recently Dillon Kearns (also of elm-radio fame), [announced a transpiler](https://discourse.elm-lang.org/t/announcing-html-to-elm-com-and-elm-review-html-to-elm/7083) that translates literal HTML code into Elm code as an elm-review fix. This is roughly how the optimiser would work, though obviously it would be doing a quite different job.

Note, that the two are quite different, the whole point of Dillon's tool is to transform your source code and then use that as the actual new source, ie. the stuff that gets checked into version control. An optimiser on the other hand wants to keep the original code, so it would need to output the optimised code in a new directory which you then compiled as part of your deployment script.

As coincidence would have it, elm-radio very recently [interviewed Evan Czaplicki](https://elm-radio.com/episode/open-source-funding/) and in it he hinted that one of the current investigative projects in the Elm compiler involves *"aggressive inlining"* something pretty close to the *'compile-time laziness'* I spoke about. In particular aggressive inlining can avoid unnecessary computation of the first argument to `Maybe.withDefault` (and similar such functions such as `Result.withDefault`).
