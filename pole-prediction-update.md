---
title: "Pole prediction update"
tags: pole-prediction
date: 2021-03-13
---

Since this blog is being hosted on poleprediction and was originally meant to document some of the effort going into pole-prediction I thought I'd give a summary of where I am.

## What is pole prediction?

This is a simple website for myself and a few friends to make predictions on the outcomes of formula one races (and qualifying sessions). The basic idea is that you guess the top ten drivers and are awarded points based on how well you've predicted them. These points are summed over the whole season. I'm the only one who has never won this.

The front-end has always been written in Elm, which is a little surprising since I was only just getting into Elm when I started with the first version. That first version had a Python backend with an SQLite database. Because serverless was quite popular at the time, and because I was hosting the front-end on 'vercel' (which was called `now` back then), I decided to try out a Javascript serverless backend with a MongoDB database. I quite liked the MongoDB database, but didn't get on terribly well with the serverless architecture for the backend for various reasons. So here I am now re-writing the front-end still in Elm, but also attempting to write the backend in Elm. It's still going to be a normal client-based single-page-application because I'd quite like to compare using Elm for the backend with Javascript.


## Status

I pretty much have all functionality available, with some issues. The missing functionality is the leaderboard. Traditionally the leaderboard was written in Elm, it just downloads all of the entries and then calculates the leaderboard to show. This probably wouldn't work well if we had many thousands of users, but we have fewer than ten. The great thing about doing the backend in Elm now, is that if I had to, I could re-write the leaderboard for the backend. 

Although all of the other functionality is technically there in the sense that you can acheive all the things you need to do not all of the checks are currently written. So currently you can place an entry for any event, even one that has already started (or even finished and has a result for that matter). Even non-admins can add/remove events, and although you couldn't do it with the front-end you could place an entry for someone you're not logged in as.

Still here is a list of the things you can do:
1. Create/edit/delete drivers
2. Create/edit/delete teams
3. Configure entrants (driver + team)
4. Create/edit/delete events, which have sessions within them
5. Create an entry/prediction for a given session within a given event
6. Input the result for a given session within a given event.

There is one purely front-end feature that I haven't done yet that was in the old version. Something I called 'motd-mode' but might rename 'highlights-mode' which hides all current results and therefore a session looks as it does during the live session. That is, you can see your own and others' predictions, but you cannot edit your prediction and you cannot see the result. This is great if you do not have the television package to watch the race live and are watching the highlights hours after the actual race. 


## Elm on the backend

So far I haven't really encountered any particular set-backs or problems. It has all been pretty straightforward. Although there is persistence on the backend, the entire 'model' is held in memory by the server, so I guess at some point there would need to be some strategy for dealing with that. This is basically a toy application so I'm to some extent allowing Elm on the backend in easy mode.

The first big test will be March the 27th, qualifying session for the first grand prix of the season.

