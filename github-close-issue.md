---
title: "Github close issue"
tags: programming
date: 2021-02-23
---

Once again today I had a github issue close accidentally on me. I'm kind of at a loss for how this feature is designed, programmers use github, we're happy being explicit, why make it *'smart'*?

So to explain the issue. Suppose you have an issue, say issue number 312. If your commit message looks to github as if it has fixed the issue then it will automatically close the issue for you. What does "looks to github has if it has fixed the issue" mean? Good question. If your commit message contains something like "close #312" or "fix #312" then that will look like it is meant to close the issue. You can also replace the "#312" part with the actual url of the issue, it seems to work even if the url targets a particular comment on the issue rather than the whole issue, which is particularly vexing.

The problem is, our issues often have separate checkboxes, and I have small commits that do one thing. But I still want to relate the commit to the message, so I have to be careful not to use the keywords 'close' or 'fix'. I'm often writing something like "Fix the login email field label, towards: #312". Unfortunately this closes the issue but that's not what I wanted at all.

Unfortunately there seems to be no way to turn this wretched feature off. It's pretty annoying to have issues closed accidentally, in fact it could be downright dangerous, the issues are there to remind us to fix things that need to be fixed. If they are being closed out from under you accidentally that is a serious problem.

What I do not understand is why on earth this feature attempts to be smart? I'd be pretty happy with this feature if I had to use specific syntax to trigger it. Suppose I had to specifically write `close(#312)`. That would be a great feature, I could mostly ignore it, because I'm quite happy deliberately closing the issue by hand, but those that like this feature can still use it, and nobody would accidentally close any issues. The worst thing that could happen is someone accidentally leaves an issue open. But that's **far** less of a problem than accidentally closing an issue.

Imagine if you wrote: "Fix the email field label of the login screen: close(#312)". Not only is that a clear instruction for the automated closer, it's **also** a clear instruction to anyone just reading the commit message.

Come on github, I'm begging you.
