---
title: "Issues in Git"
tags: programming
date: 2021-03-14
---

**Update** We have started using [Constructor](https://app.constructor.dev/) which allows us to do relative issue priority ordering. The team are very responsive to feedback as well which is great. Recommended.

For projects involving more than one person, I'm typically using github or gitlab, either has a pretty usable issues feature. For the most part with issues all I want is the ability to write some text, preferably in markdown. The ability to add screenshots is a major plus. The checkboxes are nice as well, but I'd be happy to write those myself in markdown, ie. I don't really need the ability to check a checkbox with the mouse, but it's a nice touch I guess.


What no issue tracking feature I've found has, is the ability to do relative priority. Most issue trackers allow for priority, but that is an absolute measure. Typically you can provide a priority entered as an 'integer'. But I find this mostly useless. What I really want to do is when creating a new issue, be forced to place it somewhere in the list of issues ordered by priority. Decide what issue it has higher priority than, and place it there in the list. This also means that as you close issues the priorities of the open issues are automatically updated. Obviously this does not work whenever your issue tracker is public, though even then you could have a public issue tracker and then triage those submitted into an internal priority-sorted issue tracker.


This is why for personal projects, I typically just have a text file with a list of open issues. I store this in the git repo. This also means that closing the issue can be part of the commit which deals with it. It also means that sorting an issue up or down is just a matter of cutting and pasting it either higher/lower in the todo file. I find this works pretty well, and the only thing it really lacks is the ability to add screenshots.

Indentation provides a simple way to do sub-tasks as well.  So my todo file might look something like this:


```markdown
- [x] Implement user logins
- [ ] Implement the ability to save an entry
    - [ ] Develop the backend endpoint
    - [ ] Develop the frontend UI.
    - [ ] Make sure one cannot place an entry for a closed competition
- [ ] Implement the leaderboard.
```

Typically, I cross the items off and then at somepoint have a cleanup whereby I remove all the crossed off ones, but of course you don't necessarily need to cross them off you could just delete them instead. It's still held in the source code control.

I find this works really well for personal projects, and means I do not have to leave the text editor to manage my issues. I've seen a bunch of command-line tools to manage such a todo file, but to be honest I find that just editing it directly is good enough. At one point, when working on a large project for a client, I produced a command line tool that took this as input (including the sub-tasks) and produced a nice HTML file that showed all the completed tasks and those still to do, those that were partially done (some sub-tasks completed). This was good for the client, but I think was mostly an itch I wanted to scratch. 

Anyway overall, other than the lack of screenshots, I think this method of keeping track of issues is pretty good. Of course I've mostly used this on small personal side-projects, and because they are single person side-projects there isn't much 'discussion' of an issue. I certainly have points that are not checkboxes which are usually just notes to myself, so I do think it **could** work with a team larger-than-one, but I have not tried it.


