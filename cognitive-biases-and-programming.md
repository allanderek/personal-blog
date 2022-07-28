---
title: "Cognitive Biases and Programming"
tags: programming
date: 2021-01-19
---

I've long been interested in psychology. I find it fascinating the ways in which our brain is capable of tricking even ourselves. Even if **some** results are difficult to re-produce in the real world, the fact that they work in a laboratory setting is interesting enough. I've long thought that many cognitive biases have a strong application to software development. If I had the time I'd love to create a catalog of cognitive biases and their particular application to software development. In lieu of that, I'll settle for blogging about the odd one.

Today's cognitive bias is the [IKEA effect](https://en.wikipedia.org/wiki/IKEA_effect):

> The IKEA effect is a cognitive bias in which consumers place a disproportionately high value on products they partially created. The name refers to Swedish manufacturer and furniture retailer IKEA, which sells many items of furniture that require assembly. 

I believe this can have a strong effect on software development particularly with respect to [Not invented here syndrome](https://wiki.c2.com/?NotInventedHere). I think in general programmers are a bit happier when they can justify authoring a solution themselves rather than use someone else's, even though they *know* that re-inventing the wheel is needless work. But my point here is not that *Not invented here syndrome'* exists, it is more that it is possibly exacerbated by the IKEA effect.

Once a duplicate/competing library is built in-house, the very act of having built it is likely to cloud your judgement over whether or not such an effort was worthwhile. Once you have convinved yourself that *'whilst re-using code is normally a good idea, in this one case I did the right thing by re-writing it myself'*, it becomes much easier to convince yourself that re-writing **another** library is likely to be worthwhile. This can, in theory, produce a cyclic effect, the more you write yourself, and convince yourself it was worthwhile, the more likely you are to make that same choice in future decisions. An invirtuous cycle of unnecessary work.

So you and I now both know what to do. We should consider our own work more sceptically than we are inclined to. We should also value re-using an existing library more than we do, and start each decision with the assumption that re-using the existing library is the better choice. We both know this, but I'm pretty sure I, at least, will ignore this advice at the very first opportunity. Humans are weird.
