---
title: "Relative issue prioritising"
tags: programming maintenance
date: 2021-01-29
---

**Update** We have started using [Constructor](https://app.constructor.dev/) which allows us to do exactly this. It doesn't yet have a 'public' issue board, but I believe the developers are working on it.

We are a three person team and we use github to store our source code. We also use github to track our issues. There is a missing feature that I've never seen in any bug-tracker: relative prioritising of issues. We typically have a meeting after a sprint of work to decide what to work on next. During this meeting we (perhaps after creating some new issues) mark several with the tag "priority". But what I really want to do, is *order* the issues by priority.

Now some issue-trackers do have a 'priority' field, in which you can enter a number, and then you can of course sort by priority. But this doesn't really work. What you really want to do is give a priority *relative* to the other issues. The problem with giving an absolute priority is that what do you do when you finish an issue? Bump all the issues with lower priority up one? What about when a new issue arrives? Bump all the issues considered lower-priority down lower. 

So what I really want to be able to do is order the issues by priority, and then be able to move issues up and down that list. This might seem hugely in-efficient, but relative priorities do not change **much**. They do change as time  goes by, but mostly as long as you insert a **new** issue in the correct order, then this list will take only a little maintenance.

Sometimes you might wish to move an issue, perhaps because you've realised that another higher priority issue depends upon it. Or perhaps you fix a problem that was blocking an issue, so that the blocked issue is now more of a quick-fix. Quick fixes can often be bumped up the priority list, simply because you can get rid of them.

There is of course a complicating factor that I'm happily ignoring. We are currently in Beta, which means our issue tracker is closed to the public. Only we create issues on the issue tracker. However we do have a separate issue tracker open to the public for users to create issues for us to look into. I find this solves a couple of problems. The public issue tracker acts as a kind of triage, where we decide what (if anything) to do in order to address the issue. Once that is decided we move it to the private issue tracker. If only the private issue tracker was sorted by priority we could insert the new issue in the correct place at that time. The downside is that's it's relatively easy to ignore the public issue tracker and concentrate on the private issue tracker, but there may be a critical top-priority issue in the public tracker. 

If splitting the issue tracker by public/private is not possible, another solution would be to simply default all new issues to the highest priority, with some sort of label on it meaning  "triage this and put it in the correct priority order". That would mean it gets looked at quickly.

So that is my wish, an issue tracker that allows for *relative* prioritising.

