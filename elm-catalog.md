---
title: "Elm catalog"
tags: elm 
date: 2021-01-04
---


[Alex Korban](https://korban.net/) has released another version of his [Elm Catalog](https://korban.net/elm/catalog/packages). I think this is both very important work and very well done.

This century it's become popular for languages to include a package/library manager. I'm not quite sure when this started to become popular but certainly in the 90s such package managers were not common. This lead to many different ways to install libraries, and mostly including a new dependency was a chore at best. So the introduction of package managers as part of a language's eco-system has been a major boon.

However, simply having a central place to publish libraries does not enitrely solve the issue of library discovery. I don't have any solution to this, but I'd like to distinguish two separate problems that arise. The first is the well-known and discussed problem of **ranking** libraries, which Alex Korban's catalog goes someway to addressing. The second is discovery of libraries you do not know that you need.

## Ranking

This turns out to be really difficult. There are obvious solutions to try, but most of them fall short for various reasons, though mostly it's a "first-mover" problem. To understand the first-mover problem, think of an obvious possible solution, let's try ranking packages by the number of downloads. The problem is, that the first library written will have more downloads than the second library that solves the same problem. Because it has more downloads it also *gets* more downloads, since because it is ranked first, it is downloaded and tried first. In order for a challenger library to surpass it, it must have significant benefits over the first mover.

Other possibilities basically have the same issue, you can try using github-stars, but if the first library is *satisfactory* it may well garner more stars simple because the additional libraries are never tried, even though they may, in some objective sense be better.

The same thing happens if you try to implement a manual rating by library consumers.

These kinds of rankings have a further issue. A more focused library may be better, for its particular purpose, than a library more broad in scope. However, because the more broadly scoped library is useful in more circumstances it garners more downloads, stars, votes etc.

In general ranking absolutely doesn't make sense, since you cannot always say that library A is objectively better than library B, but must instead rank whether library A is more approriate for a given circumstance than library B. Even if *typically* you may have a general preference between libraries for particular reasons.

## Discovery

Discovery though is a problem a little distinct from ranking. General purpose libraries suffer from the problem that users do not know that they need it, or less grandiosely, may benefit from it. For example, if you want to work with JWTs, you know that there may be an Elm library that helps with that, so you go to the official Elm package repository and you search something like JWT, maybe you find some libraries, maybe you have a tough time deciding which one to use, but you knew you wanted to at least try using a library. However, packages that are more general purpose it's less obvious that they might exist. A good source of such libraries are named <scope>-extra, such as [List extra](https://package.elm-lang.org/packages/elm-community/list-extra/latest/), [Html extra](https://package.elm-lang.org/packages/elm-community/html-extra/latest/), [String extra](https://package.elm-lang.org/packages/elm-community/string-extra/latest/), and [Json extra](https://package.elm-lang.org/packages/elm-community/json-extra/latest/).

The point here is that these libraries may well help in your application, but there isn't a particular pain point you come across that immediately alerts you to the fact that there might be a helpful library. So you tend to come across these libraries through happenstance, and you build up an array of useful general purpose libraries that you call upon in most non-trivial applications that you write. However it would be nice if somehow such libraries were a bit more discoverable.

One problem is that you may well introduce your own module for doing something similar. Maybe you call it `Html.Utils` in which you define `nothing = Html.text ""`. The problem is that when you do discover the [`html-extra`](https://package.elm-lang.org/packages/elm-community/html-extra/latest/) library you now have two ways in your project of doing the same thing. So you **should** go through and find all the places you've used your homebrew version and change it (and remove your homebrew) version. Sometimes this is a bit harder than that because the equivalent defined in the published library doesn't have the exact same type as your home-grown one.

## Conclusion

Anyway, discovery of general purpose libraries is of course a fairly minor pain point in comparison to the problem of ranking libraries. Nonetheless consider this post a reminder to go and check if there are any general purpose libraries that you might benefit from that you aren't using.
