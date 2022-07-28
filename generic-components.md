---
title: "Generic components: A small downside"
tags: programming
date: 2021-02-25
---

This site is the blog of a developer of a site which is a game about predicting formula one results. So it is no surprise that I might frequent the [site of Formula-E](https://www.fiaformulae.com/). What I want to talk about is evident on this site, if you go to the [standings page](https://www.fiaformulae.com/en/results/standings/driver?championship=2022020), the one linked here is specifically for the 'teams' standings, rather than driver standings, but the point is basically the same, although more pronounced because there are half as many teams as their are drivers.

On this page is a list view of the teams, it must have been made from some component. Look at the bottom of the list:

![bottom of the list of teams](/img/formulaE-teams-load-more.png)


Upon seeing this, if you're not quite into FormulaE you might wonder how many more teams there are. Let's click the button and find out:

![bottom of the list of teams](/img/formulaE-teams-two-more-teams.png)

So we got a whole two more teams. So there are twelve teams in total. That will never require a *'load more'* butotn.  Now any one who follows any kind of motorsport knows that the number of teams typically does not change during a season. It's also pretty constant across seasons, Formula 1 for example has had ten teams for a long time. So the number of teams in Formula-E will very likely never be more than 20, at least not whilst this website is still being used. You could easily just display all the 12 teams *always*, no need for any 'load more' button.

This means I highly doubt this *'feature'* was asked for. I'm not exactly sure how this has also remained in place for several years. Nor am I entirely sure how it could have come about in the first place. My guess is that whoever implements the website isn't all that interested in formula E (which is a shame because I bet there are thousands of able developers who **are** interested). That developer was simply asked for "A list of teams", and reached for a standard list component that does a bunch of things automatically for you, such as show the first 10 with a *'load more'* button.

This is one of the downsides to generic components, it means there are a bunch of decisions that you do not have to make. Of course, there are a lot of *upsides* to using generic components, most importantly there is a bunch of stuff you do not have to implement so that you can get on with implementing the stuff that is really unique to your site or application. Still, once you notice this you start seeing it in other places. To me it comes off as a lack of *craft*. Whoever is responsible for this site, just doesn't **care** enough to get this right, generic components or not.
