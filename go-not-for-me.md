---
title: "Why is Go (lang) popular?"
tags: programming
date: 2021-02-22
---

There is no denying that Go, the programming language is pretty popular. I've never understood this popularity. The most commonly cited reason appears to be that Go is simple/easy language designed for beginners. I don't feel this at all when I have had to dip into Go. This post will necessarily be quite negative for Go, but try to think of this as the writing of someone who accepts that the popularity of Go is entirely warranted, and that they just cannot see the merits.

The reason I'm writing this today is that a couple of articles popped up on my feed, the first is an article explaning why [Go is not an easy language](https://www.arp242.net/go-easy.html), most of which I agree with. The second was that apparently the Discord team are switching from [Go to rust](https://blog.discord.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f). My all-time favourite Go-critique [article, from faster-than-lime](https://fasterthanli.me/articles/i-want-off-mr-golangs-wild-ride), is long, but a really great read.

## Modern

I think this is my main gripe with Go. I don't think it's modern at all. In fact, I think it has actively tried to ignore most of what we have learnt about programming languages since at least the early nineties. To be fair, Go has garbage collection, I think we've learnt that garbage collection is a major boon to developer productivity. Joel on Software [wrote](https://www.joelonsoftware.com/2004/06/13/how-microsoft-lost-the-api-war/) that it was automated memory management that made object oriented systems popular because they increased the productivity of developers who then misattributed it to object-oriented (or something thing):

> A lot of us thought in the 1990s that the big battle would be between procedural and object oriented programming, and we thought that object oriented programming would provide a big boost in programmer productivity. I thought that, too. Some people still think that. It turns out we were wrong. Object oriented programming is handy dandy, but it’s not really the productivity booster that was promised. The real significant productivity advance we’ve had in programming has been from languages which manage memory for you automatically. 

> Whenever you hear someone bragging about how productive their language is, they’re probably getting most of that productivity from the automated memory management, even if they misattribute it.

### Pointers

So yes Go does indeed have automated memory management, so perhaps it did learn that. However, many modern languages do not distinguish call-by-value and call-by-reference. Basically every time you pass into a function/method something other than an atomic value (something other than an Int, Bool, Float, etc.), then you're passing in a pointer. In object oriented languages that is usually a pointer to an object (but might be a pointer to a string). The type system (whether static or dynamic) does the correct thing when you dereference that pointer, with something like a field access (e.g. `p.x`), but you never have to worry about that. In Go you do. For some reason.

I'm sure someone well tell me that you can occasionally be more efficient by passing some struct in by value, but I basically just don't buy that. Also I think you're bound to be less-efficient in other places by accident.


### Generics 

I might have accepted pointers, but generics is just weird that this was not part of the language from the beginning. They are such an obvious boost to architecture. Not only that, there are several languages that **didn't** include generics from the get-go which then had to retro-fit them in, usually with pretty ugly syntax. The problem with this is that the most beneficial places for generics are in the core of the standard library, so now what? You maintain two versions of the standard library to remain backwards compatible or your standard library doesn't use generics or what? That's a hard problem to solve and one that was 100% avoidable. Generics have now been accepted  into the future of Go, but they should have been there from the start. They would have been in any *'modern'* language.

### Syntax

Modern languages typically have a parsimonious syntax. Perhaps I say this with a bit of bias as Elm is one of my favourite languages and is very conservative when it comes to syntax. Other languages tend to have a lot more syntax, such as Haskell, though I'm not sure I would describe Haskell as 'modern'. Anyway additional syntax is often a sign of maturity, but it's also a sign of having tried a bunch of things that didn't really come off. You're kind of stuck still supporting that syntax because *some* libraries use it.

Anyway, I think most programmers agree that keeping syntax to a minimum is a laudable goal. But Go doesn't seem to have had that at all. The Go grammar is pretty complex for a (relatively) young and supposedly modern language. I realise this is something of a flimsy point, but it *feels* like Go has a lot of unnecessary syntax, especially for a language hoping to be simple/easy to use.

### The name 'Go'

Okay so this is obviously something of a frivolous complaint. But I think it epitomises much of what is wrong with Go, that it just doesn't seem to have learnt from any other languages. Many languages were conceived of before the internet was really ubiquitous and before it was obvious that developers would spend some percentage of their time searching the web for things related to their current issue. It's a bit easier to search for those helpful snippets if your language doesn't contain a common word that is in the solution to **other** language answers. For example, it's kind of hard to search for answers specific to the *'Clean'* programming language. The more unsual the name of your language is, the easier it is to search for. E.g. Haskell is a pretty easy language to search for. Python, Ruby are not too bad. Javascript maybe should not have called itself after Java, but it's not too bad. 

You would think that a language developed in 2009, at the most successful search engine company in the world, would have know this, and would have chosen a name easy to search for. If I had to pick a name that would make it **difficult** to search for, I'm not sure I could do **much** better than *'Go'*. I mean maybe if I really tried hard I'd name a programming language *'HTML'*, but that's kind of cheating. Maybe something like *'class'*, or *'method'*, would be worse. I doubt we'll ever know because I doubt anyone would make this mistake again.

Obviously this is a pretty trivial complain, still I feel it epitomises my complaints about Go. It **could** have been better in this regard. It wasn't a lack of luck, this information was available in 2009. The 'Clean' programming language creators were unfortunate in this regard. Similarly all the evidence that your modern language was going to need generics was there on the table. Ditto, pointers.

## My team uses Go

My team uses Go for the backend. When we discussed this choice a couple of years ago, I was not very keen. However, my colleague asked me a decent question, what language should we use instead? Many of the alternatives did have some drawbacks. I was keen to use Python, but he was worried about speed, I didn't have enough of an argument to assauge that fear. I wasn't terribly keen on the object-oriented languages either Java or C#, either of which I *could* have used, but I didn't feel would be much of an upgrade over Go. I suggested Rust and he suggested that was too much of a risk, I agreed and even [are-we-web-yet](https://www.arewewebyet.org/) kind of agreed, certainly not the resounding Yes it is today. That left functional languages, O'Caml and Haskell, were both considered. I would have taken a punt on either, but the backend was going to be mostly my colleague's concern so I let him choose Go. To be fair, I do think both of those languages would have been serious risks for various reasons. My colleague had even developed another web application using Haskell, and half-way through re-written in Go, IIRC for database reasons.
So I do see that Go occupies something of a space that means it doesn't have many direct competitors, though I believe Rust is changing that as the [above Discord article](https://blog.discord.com/why-discord-is-switching-from-go-to-rust-a190bbca2b1f) concurs with.

## A good eco-system

My best guess as to why Go is popular, is that Google have backed it in quite a large way, meaning that there are a bunch of pretty decent libraries for Go. This is similar to Java 20-25 years ago. I believe that when a language has a good set of libraries developers are productive using that language. The productivity they perceive is mostly due to the existance of professionally developed libraries which avoids them having to re-implement many things. Developers (perhaps) mistake this productivity for the **language** per se, rather than the eco-system.

I realise that this is a pretty negative conclusion, basically stating that Go is not a language that merits its popularity but has an eco-system that does. That almost sounds like sour grapes. Fair enough. I really dis-like writing a complaining or negative post about someone else's technology. The people who have made the thing you're complaining about, in this case Go, I'm sure have tried their best and they are likely (justly) proud of what they have achieved, so it doesn't sit well with me to complain. Also, there is the obvious answer to all complaints, which is simply, if you do not like it, don't use and maybe see if you can do any better yourself. So I feel bad writing it, but, I just don't like Go and struggle to understand its popularity. Sorry.

