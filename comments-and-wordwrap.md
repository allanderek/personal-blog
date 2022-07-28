---
title: "Comments and word-wrap"
tags: programming syntax 
date: 2021-01-15
---

Since almost as long as compilers have been around there have been attempts to store source code as something other than plain text. Various different formats have been suggested but the actual format is not important the general **idea** is to store the abstract syntax tree of your program rather than text that can be parsed into the abstract syntax tree of your program. There are several reasons you might wish to do this, and several disadvantages as well. But I wanted to speak today about one particular part of this. Comments.

As in many other languages comments in Elm can be written in two styles

```elm

-- A comment can be started by two dashes in which case it is ended by a new line
-- this is why you have to put the two dashes before
-- every line in
-- a multi-comment line.

{- This is the other style, it is started with a brace and dash
and continues over
as many lines

including blank ones until a closing comment token appears.
A closing comment token is a dash followed by a closing brace
-}
```
Regardless of whether you use `--`or `{- -}` **syntax** comments, there are two general *styles* of comments:

The first kind of comment is one in which the new lines are important. These are relatively rare, but do
include commenting out source code. We normally discourage commenting out source code but there are certainly
times when that is useful.
There is a second kind where the comments actually have semantic meaning, these are things like doc comments
or tests embedded in comments, or directives to linters.

Finally there is the kind of  comment I'm talking about today, one where what is written is mostly prose. You're describing some interesting part of the system or detailing why something is coded in a particular way. There are two ways in which you could write these comments, one is on a single line and just allow the editor to word wrap it on view. The second is to explicitly break up the comment into multiple lines. Note that's is explicitly not necessarily manually, the editor may well be able to insert the new lines for you. The first style is more akin to storing the abstract syntax tree, because you're not recording meaningless structure. 

In theory, because the new lines have no semantic meaning, we should put such a comment on a single line, and allow it to word wrap. But this tends to not play nicely for a couple of reasons:

1. Not all coders have their text editors set to word-wrap, which results in comments disappearing off the screen, to the right.
2. If you want to change a part of the comment, then your diff will involve the whole comment which increases the chance that you'll conflict (though people might suggest comments are never updated and definitely not often enough to worry about conflicts).

However, it means that when you **do** change part of a comment you do not need to re-justify your paragraph of a comment. Though of course a decent editor pluggin should be able to perform this task for you. Still it feels like splitting up the lines of your comment is recording accidental structure that is not important to record.

If you use a single line and allow the editor to word-wrap for display, then whenever you **do** need to split up a comment for example to take a new line to highlight a named function, or to introduce a new paragraph or something, then the new lines within your comment are actually meaningful, and what's more they will stand out all the more because it isn't commonly done.

Finally not everyone has the same width of screen. Let the user choose how wide they want their lines to be wrapped at, those with wider screens can have longer comment lines and therefore get more lines on the the screen. Of course lines can be too long such that reading the prose is difficult, but then your 'window' shouldn't be that wide otherwise you would potentially have difficulty reading the code anyway.

Why then do I never see source code comments written in this way?  I arrive at a common conclusion for me, where my preferred style, to manually word-wrap all comments by splitting them into new lines seems to be logically the worse and I cannot find a good reason to continue supporting it. But I'm going to continue doing comments like that nonetheless.
