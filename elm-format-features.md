---
title: "Elm-format features"
tags: programming elm
date: 2021-03-28
---

I've spoken [before](/posts/2021-01-27-frugal-syntax-formatter) about how [great and liberating elm-format](/posts/2021-03-25-elm-format-multiline-boolean-formulas) is. Once you cede control of the formatting of your source code, other possible features arise. Some of these could potentially take the burden away from the code editor tools. An obvious example would be the formatting of multi-line comments. When you change a multi-line comment, you often have to do a bit of formatting if you wish to keep the lines under a given character count.

To be sure, this feature would be difficult to get right, there are many edge cases. What happens if the code is already indented far to the right? What happens if the multi-line comment represents some commented-out code? However, this is just one particular possibility. The point is that once you are happy having all of your code formatted automatically, the formatter could in-theory start recognising other points within the code, and sort them for you. I mentioned commented-out code, how about it recognised that and actually formatted it for you, as normal code, but commented? What about doc-comments, these can be uniformly formatted as well. 
