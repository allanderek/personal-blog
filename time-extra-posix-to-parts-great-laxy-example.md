---
title: Laziness would be good for Time.Extra.posixToParts
description: A motivation for why laziness is a good programming language feature.
date: 2022-10-27
tags: [elm, gren, programming, laziness]
---

In this post I'm going to show a good example of where laziness would work well. This is of course not an argument
that lazy programming languages are somehow better than strictly evaluated programming languages. Rather what I wish to
do here is answer the question, what is laziness good for?


My example here comes from the [justinmimbs/time-extra Elm package](https://package.elm-lang.org/packages/justinmimbs/time-extra/latest/).
The main purpose of this library is to provide a means for working with the standard library's [Time.Posix](https://package.elm-lang.org/packages/elm/time/latest/Time#Posix)
values. Functions are provided to calculate the difference between two time values, and also to add/minus a given interval from a
given time value. So for example it provides a convenient way to take a given time value and add one day, or two hours, or six months.

However, as an additional utility it provides the [posixToParts](https://package.elm-lang.org/packages/justinmimbs/time-extra/latest/Time-Extra#posixToParts)
which takes in a `Time.Posix` value (and a zone) and gives back a record:

```elm
type alias Parts =
    { year : Int
    , month : Month
    , day : Int
    , hour : Int
    , minute : Int
    , second : Int
    , millisecond : Int
    }
```

So a common way you might use this, is to get the 'date' part of a time value. That is get the `year`, `month` and `day` values.
So suppose you have a bunch of events, all that have start-times stored as a `Time.Posix`. Now, what you might want to do is show all of
those events which are on today. Assuming that you have the current time as a `Time.Posix`, you can do something like this:

```elm
type alias Event =
    { start : Time.Posix 
    , -- Presumably other relevant data about an event.
    }
        
getTodaysEvents : Time.Zone -> Time.Posix -> List Event -> List Event
getTodaysEvents zone now events =
    let
        todaysParts : Time.Extra.Parts
        todaysParts =
            Time.Extra.posixToParts zone now
        isToday : Event -> Bool
        isToday event =
            let
                eventParts : Time.Extra.Parts
                eventParts =
                    Time.Extra.posixToParts zone event.start
            in
            eventParts.year == todaysParts.year
              && eventParts.month == todaysParts.month
              && eventParts.day == todaysParts.day
    in
    List.filter isToday events
```

This will work perfectly well, but it's doing quite a lot of work that is unnecessary. For each event it is calculating not just the,
year, month, and day associated with the start `Time.Posix`, but also the hour, minute, second, and millisecond. These values are
calculated, but just thrown-away. If there are a lot of events, then it might be desirable to write our own version of `Time.Extra.posixToParts`:


```elm
type alias Date =
    { year : Int
    , month : Time.Month
    , day : Int
    }

posixToDate : Time.Zone -> Time.Posix -> 
posixToDate zone time =
    { year = Time.getYear zone time
    , month = Time.getMonth zone month
    , day = Time.getDay zone month
    }
        
getTodaysEvents : Time.Zone -> Time.Posix -> List Event -> List Event
getTodaysEvents zone now events =
    let
        today : Date
        today =
            posixToDate zone now
        isToday : Event -> Bool
        isToday event =
            (posixToDate event.start == today)
    in
    List.filter isToday events
```

This works well enough, but it's slightly unsatisfying that I've had to re-implement a library function just because the
library function did too **much** work.

But note, that even this is doing potentially too much work, if the year is not correct, we needn't check the month nor day.


```elm
getTodaysEvents : Time.Zone -> Time.Posix -> List Event -> List Event
getTodaysEvents zone now events =
    let
        todayDay : Int
        todayDay =
            Time.getDay zone now
        todayMonth : Time.Month
        todayMonth =
            Time.getMonth zone now
        
        todayYear : Int
        todayYear =
            Time.getYear zone now
        isToday : Event -> Bool
        isToday event =
            Time.getYear zone event.start  == todayYear
                && Time.getMonth zone event.start == todayMonth
                && Time.getDay zone event.start == todayDay
    in
    List.filter isToday events
```

Because `&&` does not evaluate the right-hand side if the left-hand side is `False` we avoid calculating the month and day of an
event's start time if the year is not the same as the current one.

Even this version potentially does a small amount of work that it needn't. If **none** of the event start times are in the correct year
then we will needlessly calculate today's month and day. Of course in this case, that's a minor extra calculation (and probably faster than
the lazy version if laziness is done via thunks). However, note that it's non-trivial to see when you might be doing unnecessary work, and
remember, that this is pretty simple exmaple.

It's not difficult to find such cases in your own code, where either you're potentially doing more work than is necessary, or you're
crafting conditional evaluation in order to avoid unnecessary calculation. These conditionals will make your code more complex and
hence more diffcult to maintain.

A thought experiment, how could the `justinmimbs/time-extra` Elm package achieve this configurability? First of all it could attempt
having multiple functions which get different 'parts', just as our `posixToDate` function got the parts we needed. Note though that
that was still insufficiently lazy, and the number of different functions is combinatorial, though in this particular case you could
probably guess at a few common ones, such as 'date', or 'time of day'.

Of course it could return a lambda for each field of the record, but then you might evaluate that lambda more than once. In any case
if it did this it would be re-implementing laziness. 

The main point I'm making here is not that laziness is more efficient, it's that it frees you from having to consider unnecessary work
and as such can lead to simplier code structure, that is easier to maintain.
