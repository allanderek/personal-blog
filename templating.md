---
title: "Templating HTML and String interpolation"
tags: programming elm
date: 2021-04-06
---

When I first started writing web applications I was using Python, and pretty much by default I started using a templating language to actually render the HTML. There are quite a number of templating languages (or frameworks) for Python, I tried a few and found them to be mostly much of a muchness, that is to say I couldn't quite work out why so many existed since the difference between them seemed minimal to me. 

Anyway, even then I started attempting to generate HTML using primitives of the language. So when I encountered Elm, it seemed very natural to me and I took to it pretty much immediately. I think this represents something of a dichotomy, some programmers immediately like the way Elm generates HTML without templating others dislike it at first, or even put up with it indefinitely.

I keep coming across Ocaml and ReasonML. Now I was a big fan of O'caml, the compiler I wrote for my PhD thesis was written in O'caml, so I'm generally quite open to web frameworks and other such things written in O'caml. The latest (to come across my feeds) is the [Dream web framework](https://aantron.github.io/dream/). Everytime I come across such a web framework, I get a little excited, then click through, and remember that ReasonML is a kind of quasi-templating language and my enthusiasm dips a little. Here is the example shown on the home page of the Dream web framework linked above:

```ocaml
let hello who =
  <html>
    <body>
      <h1>Hello, <%s who %>!</h1>
    </body>
  </html>
```

Okay so in Elm we never directly write the `html` and `body` tags so let's make this a bit more realistic before we translate it into Elm code:


```ocaml
let hello who =
  <article>
    <section>
      <h1>Hello, <%s who %>!</h1>
    </section>
  </article>
```

Notice the sort of string interpolation going on. In most languages string-interpolation is sort of a half-way house between templating and not. I generally think string interpolation is a good feature in a language. Anyway here it's closer to templating, though because it's such a simple example (as you would expect as the *first* thing on the home page) it's basically just string interpolation. 

Anyway, here is the same snippet translated into Elm:


```elm
hello who =
    Html.article
        []
        [ Html.section
            []
            [ Html.h2
                [ Html.text "Hello, "
                , Html.text who
                , Html.text "!"
                ]
            ]
        ]
```

I agree that this is a little more noisy than the above, but I prefer it. When more realistic examples are done the noise of the Elm solution becomes less in comparison with the similar noise in the templated solution.

I've not really missed string interpolation in Elm much. I still think string-interpolation is a pretty nice feature, but I can also see that keeping the language small is a pretty good counter-weight. In addition, pretty decent string-interpolation can be [done as a library in Elm](https://package.elm-lang.org/packages/lukewestby/elm-string-interpolate/latest/) and yet I've never really been bothered enough to install such a library. This kind of suggests we do not really string interpolation.

I think the above example also highlights why string interpolation isn't really missed when using Elm. That is because you can just use multiple `Html.text` nodes as in the above example. 
