---
title: "Html over the wire"
tags: programming 
date: 2021-03-04
---

A while back a colleague and I started developing a Go based web framework, in which all the HTML was generated on the server. The client webpage was essentially a thin client that sent all messages, via a websocket, to the server which then returned the new HTML to be displayed. I had previously helped with a very similar technique done in Python. This general technique is being experimented with in various places, most notably in [Phoenix Live View](https://www.phoenixframework.org/blog/tags/liveview) (an Elixir based effort), and [Hotwire](https://hotwire.dev/).

This approach is pretty exciting, the best part is that it takes the problem of knowing what data you need to show the current page and moves it to the server, where typically you can just grab the data you need. So for example in a single-page-application, if the user navigates to a new route, you might need new data to show that page. In a traditional client-side rendering approach, you would have to work out what data you require and make such a request. In the server-side approach, you would potentially have access to the database **whilst** you're rendering the page, and hence that removes a fairly large chunk of complexity.

A further major advantage is that this setup allows you to push updates from the server to the client. When developing our solution to this we developed a simple two-player game, where each player was just automatically updated with the opponent's moves. This could be a major boon for things like e-commerce stores that need to keep the cart in sync across devices and tabs etc. 

One negative is that you're now doing a fair bit of rendering on the server, rather than pushing that out to your clients. So that does potentially require more resources.

Another point is that you still have some complexity here, what happens when the user clicks a button? Hopefully that request gets sent over the websocket to the server and some updated HTML returns pretty snappily. But what do you do if it's not snappily? You potentially need some kind of 'working' indicator on the client, but unfortunately you have pushed all the rendering to the server, so how can the client display a *working indicator*? You could have the client display a **general** working indicator whilst **any** message is in flight. That's going to be a major pain if you end up sending a message for, say scrolling, because you want to 'record' scroll positions etc. Remember a general working indicator would have to do something like shim the entire screen with an indicator in the middle.

Okay so you can probably split up your messages a little, here are messages that the user needs to wait on being responded to, and here are other messages that the user doesn't need to wait on. Or you could possibly have some kind of method of determining from which element a message was sent. Eg. a *'submit'* message could be sent from the entire `form` element thus disabling it until the message returns. You can also take a *'wait for a quick response'* approach, in which you send the message and then wait around a quarter of a second for a response, if none has returned by then, you show the working indicator. If you tweak the timing correctly you won't get continually 'flashed' with working indicators. Hopefully all of this complexity, because it is quite general, is in the framework you're using.

Overall I definitely think this scheme has potential and is a really interesting area of innovation. I suspect there are some more complexities than are initially obvious and for that reason it's not *quite* the dream like development scenario that is being painted. In particular, look at [hotwire's homepage](https://hotwire.dev/), it states (my emphasis):

> This makes for fast first-load pages, keeps template rendering on the server, and allows for a **simpler, more productive development experience** in any programming language, without sacrificing any of the speed or responsiveness associated with a traditional single-page application. 

This sounds like there isn't much of a learning curve and everything is simple. But read further down the page, you learn that the heart of *Hotwire* is *Turbo*, fair enough, but that's not apparently enough because you also need something called *Stimulus* which:

> While Turbo usually takes care of at least 80% of the interactivity that traditionally would have required JavaScript, there are still cases where a dash of custom code is required. 

So here I think some of the hidden complexity is creeping in. But even *Stimulus* is not enough, you also will need something called *Strada* which:

> Standardizes the way that web and native parts of a mobile hybrid application talk to each other via HTML bridge attributes. This makes it easy to progressively level-up web interactions with native replacements. 

At this point things have become a bit more complex for me to fully understand exactly what is going on. I didn't realise there was going to be any need for *bridge attributes*. That of course looks like you're only going to need that if you're making a *"mobile hybrid application"*.

Now it could be that a lot of this complexity is going to be hidden by the framework, in this case Hotwire. As I've said, I'm very excited by this approach. I'm just not quite believing the full hype just yet, since I think whilst this approach will remove a lot of the complexities around developing an SPA in the traditional way, it will replace it with some complexities that aren't currently there. I suspect, at least in some scenarios, that this will end up being a positive trade-off. 

