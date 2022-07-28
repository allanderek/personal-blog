---
title: "Keywords in a language"
tags: programming
date: 2021-03-22
---

I was reading, briefly, [the ecosystem of the Go programming language](https://henvic.dev/posts/go/), mid-way through it states:

> A simple language: Consulting the spec, you can find out that Go has only 25 reserved keywords.

Which got me first wondering whether that was a lot or not, a brief search and I found a [bunch](https://stackoverflow.com/questions/4980766/reserved-keywords-count-by-programming-language) of [places](https://www.reddit.com/r/ProgrammerHumor/comments/6inj17/languages_by_number_of_keywords/) offering [keyword counts](https://github.com/leighmcculloch/keywords).

They're not completely accurate, one place has Haskell in with 25 and another 55, but for the most part sound about right. One place has Elm with 25, I make it a few less, since that seems to be based on the exports from the `Parse.Keyword` module in the compiler, which has four or five exports which are not keywords. Nonetheless I was kind of surprised that Go and Elm were roughly on a par here, and indeed 25 is comparatively fewer than most languages.

Of course the next question is, is this at all a useful measure? It seems some people think that it is, since this measure gets trotted out quite frequently (admittedly from languages with low counts as in the example above). This then tends to get [reasoned into a good measure of programming language complexity](https://richardeng.medium.com/how-to-measure-programming-language-complexity-afe4f7e75786).

It certainly seems like there is some kernel of truth here:

> [a good measure of language complexity is] the number of keywords or reserved words in the language. This corresponds roughly to the number of language features and, hence, the size of the language.

It's obvious that it's not a particularly *accurate* measure of complexity in a language. Some features of the language require more than one keyword, `if-then-else` is probably best thought of as **one** feature, but requires three keywords, `try-catch` is probably one feature, but then maybe `finally` is an additional feature?

It's also obvious that some languages are quite wordy where as others are prone more to operators. Python uses `and`/`or` keywords where most other languages use `&&` and `||` operators. Some have `True` and `False` as keywords, other have them as defined values. Other languages have keywords for pretty seldom used cases, unsafe code, or embedding assembly code. I suppose technically in those cases the language is a bit more complex for it, but if you came across such code you would have to search for what it means, in the same way as you would in Haskell for `unsafePerformIO`, which isn't a keyword. Basically many keywords are just reserved value or type names.

So I don't have any conclusion here, maybe the number of keywords in a language is a useful measure of complexity, my gut feeling is that it's just easy enough to measure and doesn't really say anything particularly important about the language. I don't have a better alternative, maybe a good place to look is the standard library, how difficult is that to follow? Hardly easy to measure.


