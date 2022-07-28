---
title: "Clients as a pitfall"
tags: elm programming
date: 2021-03-11
---

Back looking Paul Graham's old posts from 20 years ago, and back to the [language design one](http://www.paulgraham.com/langdes.html). There is a section on pitfalls and the first pitfall is about clients:

> This is just a guess, but my guess is that the winning model for most applications will be purely server-based. Designing software that works on the assumption that everyone will have your client is like designing a society on the assumption that everyone will just be honest. It would certainly be convenient, but you have to assume it will never happen.

So what he didn't really see coming was that the client would be built in the web browser and basically yes everyone would have your client, because it would be downloaded afresh each time they visit your webpage.

> I think there will be a proliferation of devices that have some kind of Web access, and all you'll be able to assume about them is that they can support simple html and forms. Will you have a browser on your cell phone? Will there be a phone in your palm pilot? Will your blackberry get a bigger screen? Will you be able to browse the Web on your gameboy? Your watch? I don't know. And I don't have to know if I bet on everything just being on the server. It's just so much more robust to have all the brains on the server.

So he also failed to see the arrival of the app-store. Nevertheless I mostly think the app-store has been a backwards step for most applications because it's simply meant that you had to develop multiple code-bases for, at least, iOS, Android, and the catch all solution the web. Many have started just going for the catch all solution and then using some wrapper to provide what seems like an iOS or Android app, but is really a web app.

So I think for the past twenty years this prediction hasn't done too well as more and more computation has been shunted off the server and onto the client. I've written about how this has the problem that you end up kind of [re-implementing the browser](https://clouddev.pakk.io:4014/posts/2021-02-12-reimplement-the-browser) in the browser.

Recently however with the likes of Phoenix Live View and Hotwire, there has been some movements to putting more of this stuff back on the server. So perhaps this prediction's time has come. I'm quite excited by these live views, though I am cautious that once again we will end-up re-implementing parts of the browser. I also think there are significant pitfalls around, but if I absolutely had to bet (and thankfully I do not) I think I'd bet on these 'live-view' style frameworks. Being able to control the brains on the server **and** being able to push changes to the client **and** being able to synchronise multiple clients' views will be pretty big wins.
