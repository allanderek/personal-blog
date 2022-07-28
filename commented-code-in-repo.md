---
title: "Commented code in the repo"
tags: programming
date: 2021-03-19
---

When first learning source code control, one of the mantras you will hear is:

> do not check commented out code into the repository

I'm not sure why this is so loudly and vehemently exclaimed. I'm not even sure it should be a general rule. I realise that the code you are commenting out is available in the repo should you need it. But if so, you at least need to add a comment suggesting that some code that would otherwise be right here and commented out is available in the repository history, where to find it, and what it does. I also see that it adds to the general untidyness. But having said that, I just don't see that it is so very harmful. On top of that there are clearly cases in which the rule should be broken for specific reasons, and I fear making such a vehement and strict rule against this will lead to it not happening. 

I'd be happy with a strict rule that stated:

> There should be no commented out code at any time without a leading explanation of why the code is commented out.

I follow this rule quite strictly, so even if I'm commenting a line out for testing/discovery/debugging I still put some kind of explanation of why it is commented out. In this way, I'm not tying the rule to the source code control system, since it doesn't really have anything to do with that.


