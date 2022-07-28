---
title: "Pole prediction lives"
tags: programming
date: 2021-03-26
---

So poleprediction is live, and not with too much time to spare since the first session of the new season was today with the first two practice sessions for the Bahrain grand prix. I technically didn't need to be ready until tomorrow since that represents the first session for which we are predicting.

Mostly everything has gone through without a hitch. A small problem with emailing, in which I cannot seem to validate my domain with mailgun, so I'm still using the sandbox which means I can only send to 5 pre-verified recipients. In addition one of them is, for some reason, not receiving the emails promptly, I suspect the issue is with Yahoo rather than mailgun, but I do not know.

I also had a bit of a hitch in that the season predictions had the wrong sign on the check for late entry. Not quite sure how that could have gotten through my testing but it somehow did. I fixed that immediately, thankfully just being a very simple case swap.

One other problem, logging in on a new device, doesn't remember your previously made guesses. Actually it's logging in anywhere, but since you'd need to logout it has only come up when moving to a new device. The issue is because the current user's guesses are downloaded on startup, but should **also** be downloaded when the user logs-in successfully. Whoops. I'm sure I'll managed to fix that this evening or over the weekend. My fellow poleprediction players have been very patient.

So, so far, all is working with an Elm backend. We'll see if perhaps the number of database messages just gets too large, over the course of the season. That's my major worry.
