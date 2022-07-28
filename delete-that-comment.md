---
title: "Delete this comment"
tags: programming
date: 2021-01-11
---

This morning someone posted their [snake game implementation](https://www.eatsomecode.com/games/double-snake/) to the [Elm discourse](https://discourse.elm-lang.org/t/double-snake-elm-game/6737) asking for feedback on their implemenation. I thought the implementation was pretty fine, I had one structural comment but for the most part I'd be pretty happy to have to work on something from that code. My complaint though was that the author's source code comments were mostly pointless, at least none of them were (yet) downright counter-productive, but it's easy enough for comments to become so. I was reminded of an excellent student of mine who wrote a blog post (as part of their coursework) entitled 'Delete this comment' you can [find it here](http://blog.inf.ed.ac.uk/sapm/2014/02/21/delete-this-comment/).


I think it's an excellent read, the basic idea is that for the **most** part comments in source code can be seen as a 'todo', ie. 'TODO: Refactor this code so that this comments is no longer helpful/needed'. I think that some significant proportion of comments can be removed by simply renaming the entity that is being commented on. Here is an example from the above snake game:

```elm
{-| Model when starting OR re-starting the game
-}
getDefaultModel : Game -> Model
getDefaultModel game =
```

How about just renaming this to `resetGame`? The type already tells you that you are returning a `Model` but if you prefer `resetModelForGameStart`. 

The original blog post concedes there are times when a comment is unavoidable. A common one is where a refactor is not possible because of code which is depended upon. For example you might comment on what looks like a strange call of an api function, to say that you are in someway working around a deficiency in the api. Sometimes the commment is indeed a TODO asking for a refactor, but you're clear about that and you're deliberately taking on some technical debt for some rational reason.

Anyway the take away here is, whenever you see a comment in your code, or at about to write one, consider whether a refactor could make the comment redundant. See comments like dead or ugly code, something that should be purged if at all possible. Many times you will be surprised that a simple re-name or abstraction will make the comment unnecessary.

