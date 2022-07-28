---
title: "Uses and definitions"
tags: elm programming
date: 2021-04-03
---

Sometime in the early part of the century, think about 2004, I saw a talk which analysed source code for defects. They had used, what was at the time fairly new, online source code repositories for open source programs. They determined that a piece of code was buggy if it was changed in a commit which fixed a bug. The authors analysed the code (which was all written in Haskell) to see if they could determine differences between *'buggy'* code and *'non-buggy code'*. I wish I could remember the authors or the title, but I cannot.

One of their results was that a use of a variable was more likely to be in error, the further it was from the definition of that variable. Again, I cannot remember the source talk, and I cannot remember how convincing the evidence or the argument was for this. But it must have been pretty convincing at the time since it's the one thing I remember from that talk and I still remember it now the better part of two decades later.

I use this to think about re-ordering definitions in `let-in` expressions. In particular I often end up with a final container (say an `Html.div`) with more or less just names:

```elm
let
    ...
    secondPart_A = 
        ...

    secondPart_B = 
        ...
    secondPart_C = 
        ...
    secondPart =
        Html.div
            [ Attributes.id "second-part" ]
            [ secondPart_A
            , secondPart_B
            , secondPart_C
            ]
    ...
in
Html.section
    []
    [ heading
    , firstPart
    , secondPart
    , thirdPart
    , footer
    ]
```

Why? Well, if instead I put half of the definition of `secondPart` inline within the final expression such as:


```diff
-    secondPart =
-        Html.div
-            [ Attributes.id "second-part" ]
-            [ secondPart_A
-            , secondPart_B
-            , secondPart_C
-            ]
Html.section
    []
    [ heading
    , firstPart
-    , secondPart
+    , Html.div
+       [ Attributes.id "second-part" ]
+       [ secondPart_A
+       , secondPart_B
+       , secondPart_C
+       ]
    , thirdPart
    , footer
    ]
```

Now the uses of `secondPart_A`, `secondPart_B`, and `secondPart_C` are much further way from their definitions than they were before. I could move those definitions down, but I can only do that for one of these parts.

Anyway the point is when I'm trying to make code tidying decisions like these, one of my metrics for deciding which is better, is which keeps my variable uses closer to their definitions. In addition, where you have some large section like this, you unavoidably have some variables used quite far from their definitions. However, if you keep such uses trivial they are far less likely to be in error. So notice in my first version, `heading` is necessarily far away from its definition, but the usage is trivial and highly unlikely to be in error.
