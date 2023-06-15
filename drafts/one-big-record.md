---
title: One big record
description: Some thoughts on structuring your data
date: 2022-09-08
tags: [elm, programming, program structure]
---

Back when IMAP first became an e-mail standard the thought was that we would and sort all our emails into separate folders.
It turned out that it was difficult to correctly structure your folders in advance, and that was worked better was to keep all emails in one big pile, but assign zero or more labels to each e-mail.
That way you could have an email that is both about say, work and golf, or family and money. If you wanted to see all your money emails you filtered to see all emails with the 'money' label.
This worked out a lot better for two reasons, one is that it simply is the case that a single email could fit into multiple categories, and the other is that you didn't need to pre-decide the structure of your inbox folders.
Should you have a money folder, with a two sub-folders for golf and family, or should you have a golf folder with two sub-folders for work and family.
With labels, you don't need to make such decisions, you just label each email appropriately, and potentially have some filters to do that automatically.

Elm programs have a large central data structure called the `Model`. It's very common for people new to Elm, though not new to programming in general, to be concerned about a `Model` type where everything is stored as one big record.
The alternative is that you structure this a bit more, so rather than your `Model` type, having say, 30 sub-fields, it instead might have 5 sub-fields each of which is a record itself with (on average) six sub-fields.

Elm also has a `Msg` type, which is typically a custom union type. Similarly it can seem unorganised to have all the messages in one large custom union type rather than structure it a bit more, with similar, or related messages grouped together.

In this post, I want to explain why such structuring of types is attractive, but then argue against. This obviously isn't a blanket argument, I'm not suggesting it's never a good idea to structure your types more, I'm merely presenting the case not to do this by default.

