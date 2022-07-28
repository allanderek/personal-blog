---
title: "Remote pairing software"
tags: programming
date: 2021-03-29
---

Pair programming is pretty popular and I'm a fan of it myself. It certainly feels as if there is a productivity boost from pair-programming, but I'm not entirely sure why. I have two theories:

1. It's not actually more productive it just feels more productive
2. Pair programming actually just forces both of you to focus for the entire time you're pairing.

I tend towards the second explanation. I've paired sometimes for three hours, even if two people pairing are no more productive than a **single** developer, I suspect you might struggle to actually get those two developers to independently work solidly for more than an hour and half each, within those three hours. There is probably some combination of factors going on here. So maybe for example two programmers pairing are no more or less productive than two developers working independently, but pairing just forces both of them to focus on development for that period of time. Additionally, it's more difficult to interrupt two developers pairing than it is to do so for either one independently developing.

Possibly all of this could be summed up by saying it's easier to get into, and stay in, the 'zone' when pairing.


Anyway, now being mostly a remote developer, I've seen quite a few solutions for remote-pairing. Previously, when teaching at the university and supervising final year and masters students' projects, there were many that purported to allow for remote pairing for some particular task, eg. modelling. I have yet to be really convinced by any of these solutions. Not because I do not think that remote pairing works, I do, but because there is an existing solution that I've yet to see any custom solution do better than. The existing solution is a simple screen share. This basically does pretty much a 100% of what you need for remote sharing, and can even be a bit better than custom solutions.

Think about non-remote pairing. Usually you have one of you control the keyboard and mouse and actually doing the typing, whilst the other thinks (out-loud) of larger structure. Remote screen-share works really well for this. In fact, since it completely takes away the possibility for the non-driver to take-over, I think it might actually be pretty beneficial. It's just another decision, which isn't all that important, taken away from you, and I think that probably helps the two participants stay in the zone.

Custom remote-pairing solutions, often provide the ability to 'hand over control'. I would suggest that if this were really necessary, perhaps you're moving on to a new block of work, such that you can commit the current block of work, and 'hand over control' by simply changing which person is sharing their screen. Because there is a *bit* of friction in doing this, you won't do it often, and I think it's really very rarely necessary or beneficial.

Of all the student projects I saw, all of them without exception focused on the technical challenges of providing remote-pairing, mostly they concerned themselves with either concurrent edits, or editing-mutual-exclusion. None of them did the sensible thing and actually compared their solution to simple screen sharing. It was a question I asked in every single project demonstration and never once did I get a convincing answer.
