---
title: "Less is more security (sometimes)"
tags: security
date: 2021-01-08
---

We have an application that uses a login token which expires. This is for an admin panel. The login token expires after a number of hours, after which you have to re-login. Initially this expiration time was set to 8 hours.
I lobbied for this to be changed to 12 hours, but not just because it is more convenient, also because I think it might increase security, or at least it's not clear it reduces security. 

Here was my reasoning. A typical work day is around 8 hours, from start to finish, something like 9 to 5. A typical admin user of our application is expected to pretty much use it all day, so it wouldn't be uncommon for them to login more or less immediately after starting work for the day. Now if they work exactly 8 hours, then all is fine, the token will expire (and if still running the application will log them out) more or less immediately after the person leaves work. But work tends to be more non-deterministic than that, so if they work say, an extra half-hour, there is a good chance that the system will require them to re-login approximately half an hour before they leave work. This means the token will be valid, but not being used, for approximately 7 and half hours.

By increasing the time to expiration of the token from 8 to 12 hours, it means that they are likely to leave work with some time remaining on the token, but less than 7 and a half hours. So I *think* this is a case where making a decision that seems to reduce security actually makes it slightly tighter. At the very least this change makes it much more convenient for the user without significantly increasing any risk. It's particularly annoying for the user to have to log in, only to complete less than 30 minutes work.

A final point, if your security is indeed annoying for the user, experience suggests that they will find ways to make it less annoying. These methods almost always make the system less secure. So if you can make a system more convenient without significantly reducing security, that may well make the system more secure in the long-term, albeit in a way you'll never know about, since you'll never discover the ways in which the users would have subverted your (inconvenient) security. 
