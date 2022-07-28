---
title: "Significant Whitespace"
tags: programming syntax
date: 2021-02-28
---

I will admit that before I tried my first programming language that used significant whitespace, I thought it would be horrible. I cannot remember why. I was thinking about this whilst reading [An incomplete list of complaints about real code](https://rachelbythebay.com/w/2021/02/22/lang/). In this the author doesn't like signficant white space, here is what they say:

> I don't like meaningful whitespace. If a tab means one thing and a space means another and they both print the same way (i.e., NOTHING, a big blank space on the screen), I'm not going to be happy. Also, if the amount of whitespace somehow controls branching or blocking of code, you better believe I'm not going to be happy when it trips someone up. 

I'm not going to argue much against this, because that's not really the point of this post, but I cannot help myself. The first complaint is not really about significant whitespace but about tabs/spaces meaning different things but looking the same. The tabs-vs-spaces debate is kind of interesting in that both sides make reasonable points, but I think Elm gets this correct, it just entirely disallows tabs. That takes the whole debate off the table. There are certainly some downsides to this, but I think the upsides are a clear win. The second point I wish the author had expanded upon, they do not make any sort of argument here, I agree I'd be unhappy if whitespace "trips someone up" but isn't it vastly more likely that non-significant whitespace is going to trip someone up?

Anyway the point I wanted to make today was about how tooling can mitigate the debate around syntax quite significantly. An example of this from long ago is line-versus-block comments. In Elm you can do line comments with `--` and block comments with `{- .. -}`. Now, think back to a time before syntax highlighting etc. when editors were just text editors rather than the IDEs they are today. These two styles of comments had complementary advantages.

* Advantage of line comments: It's easy to see if the some line is part of a comment. In particular if you comment out a block of code, that means placing `--` at the start of each line. So you can easily tell whether a line of code has been commented out.

* Advantage of block comments: You can comment out a block of code really easily.

Now editors do syntax highlighting so that pretty much removes the advantage of line comments, you can see by the colour whether a particular line has been commented out or not. However, editors can now also automatically comment a block of code, so that pretty much removes the advantage of block comments. So now if you ask me which I prefer, I basically do not have a strong opinion.


Now let's think back to significant whitespace. I think this is a major boon for programming, it means you will never have misleading indentation, and also you can reduce syntax noise. However, as [discussed on elm-radio](https://elm-radio.com/episode/elm-format/) Elm has a quite brilliant automatic formatter. It's quite brilliant for a few reasons but it is ably helped by the frugality of Elm's syntax. The best thing about elm-format is that it removes 99% of your formatting decisions from you. This is so liberating and kind of hard to describe until you try it. But my experience is such that I would find it awkward to go back to a developing without an automated formatter, and worse, having discussions with my team on formatting. 

Anyway, so elm-format is great, and once you have that, it doesn't really matter so much whether white-space is signficant or not. You will still never have misleading whitespace. I would still rather have significant white-space for two reasons:
1. It still reduces the noise of the syntax, that is, fewer parenthetical tokens required such as `{`, `}`, `begin`, and `end`.
2. I still think it's possible to briefly mislead yourself with whitespace until the formatter kicks in, but you might not notice immediately. That means there is still a bug in there until someone notices. It's more likely to get noticed now that formatter has done its thing, but still.

So the classic example of misleading whitespace is something like:

```c
    ...
    if (some_condition)
        x = 0;
        y = 0;
    ...
```
Here, you meant to reset to zero both `x` and `y` only if `some_condition` is true. Unfortunately because you didn't use curly-braces only the statement `x = 0;` is actually guarded by the conditional. I could see someone writing this and then continuing another block of code before finally saving and the auto-formatter change this to:

```c
    ...
    if (some_condition)
        x = 0;
    y = 0;
    ...
```

Or perhaps even also adding in the curly-braces. The point is, unless you're going back over all your code after you save it, the auto-formatter hasn't prevented the initial bug. This isn't a fault of the auto-formatter. It's mostly the fault of poor syntax choices. But the point is that even with a good auto-formatter I still slightly prefer significantly whitespace.

But my **main** point, is that the auto-formatter does at least somewhat mitigate this choice. Because of the auto-formatter I'm way less bothered whether whitespace is significant or not. I think I'd rather work in a language that insists on the use of an auto-formatter than one which uses significant whitespace. In other words if I had to choose between significant whitespace or a non-optional auto-formatter, I'd choose a non-optional auto-formattter. Note: in Elm the auto-formatter is optional, but it's generally expected by the community, so it's not technically obligatory but it's socially obligatory.

To wrap back round to tabs-vs-spaces, I could deal with either tabs being disallowed as in Elm, or the use of tabs being mandatory for indentation (ie. you couldn't use spaces at the start of a line). Either would be fine with me, it's the mixing of the two that is what causes so much pain. You see this time and time again, the less choice you give to programmers, *typically*, the better.
