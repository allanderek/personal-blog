---
title: "Elm and comma separated lists"
tags: elm syntax 
date: 2021-01-13
---

In Elm comma separated lists (and records and tuples) are traditionally written with the comma on the next line preceding the next item like:

```elm
elements : List (Html msg)
elements =
    [ title
    , introduction
    , mainImage
    , mainContent
    , conclusion
    ]
```

I'm quite used to this style from my Haskell days. Although the comma-at-the-start feels a bit unnatural at first, I think after a while you start to see the benefit of having a nice visual clue which distinguishes a contination of a previous item with the start of a new item. Another benefit in my mind is that it means the items naturally align even though each line has the same indentation. 

Another purported benefit is fewer edits when you move or re-order the lines. In particular if you remove the final item you do not need to change the penultimate one. This has source code control benefits in particular you're slightly less likely to conflict with a collaborator since if they edit the penultimate item whilst you delete the final one, your edits will not conflict. This is not true if you use trailing commas (in Elm).

Python gets around this problem by allowing you to follow the final item with a comma, [here's a nice video](https://www.youtube.com/watch?v=2ct_kO3mCDc) by [Miguel Grinberg](https://blog.miguelgrinberg.com/), detailing the benefits of the optional trailing comma. But the idea is the same, for example, if you write the above as:


```elm
elements : List (Html msg)
elements =
    [ title,
    introduction, 
    mainImage,
    mainContent,
    conclusion,
    ]
```

In this style (which is not valid Elm because Elm does not allow an optional trailing comma like Python), you can also re-order the lines and delete the final one whilst only having to edit the lines in question. Note I prefer Elm's style here because in this style the items are either not aligned (as in the the 'i' of 'introduction' is not directly below the 't' of 'title') or all of the lines are indented two further in than the first line.

An issue I have with both styles is that if you want to delete the **first** item, or re-order so that the first item is moved down the list, you're forced into more edits than you might otherwise need. Python's trailing comma approach can be fixed by taking a new line before the first element:


```elm
elements : List (Html msg)
elements =
    [ 
        title,
        introduction, 
        mainImage,
        mainContent,
        conclusion,
    ]
```

If Elm were to allow an optional **preceding** comma, then it could be similarly fixed as:


```elm
elements : List (Html msg)
elements =
    [ 
    , title
    , introduction
    , mainImage
    , mainContent
    , conclusion
    ]
```

Similarly to the situation I [described yesterday](/posts/2021-01-12-elm-indent-two-four/) although I've sort of ended up arguing for this style I cannot quite make myself prefer it.

