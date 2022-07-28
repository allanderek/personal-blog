---
title: "Indent with two or four spaces"
tags: elm syntax 
date: 2021-01-12
---

I've always indented Elm with four spaces. To my mind this looks nicer than two or eight spaces. It also seems to be about the default, relatively common in code seen in the wild. Recently I've found a really good reason to prefer two spaces, but I still cannot quite make myself accept it. I thought I would detail it here anyway.

The *'problem'* with four spaces comes when you want to indent nested comma separated entities. So a list of lists, or a list of record types. Suppose you have a list of people:

```elm
people : List People
people =
    [ { name : "Ringo Starr"
      , yob : 1940
      }
    , { name : "John Lennon"
      , yob : 1940
      }
    , { name : "Paul McCartney"
      , yob : 1942
      }
    , { name : "George Harrison"
      , yob : 1943
      }
    ]
```

So the problem is that when you indent to continue the *record* expression which is the first element in the containing *list* expression, you naturally indent by 2 spaces, because then the `,` of the line `, yob 1940` lines up with the with the opening brace of `{ name : "Ringo Starr"`. This means that you now have inconsistent indentation increases, where most of them are 'four' spaces but in the special case that you're continuuing a nested comma-separated entity you increase the indent by two spaces. This problem goes away if you just always indent by two spaces:

```elm
people : List People
people =
  [ { name : "Ringo Starr"
    , yob : 1940
    }
  , { name : "John Lennon"
    , yob : 1940
    }
  , { name : "Paul McCartney"
    , yob : 1942
    }
  , { name : "George Harrison"
    , yob : 1943
    }
  ]
```

My editor (nvim) has a nice increase/decrease indentation command (`<<` and `>>` in normal mode), it even nicely 'snaps' to a multiple of your chosen indent (so for me normally 4), so this situation kind of breaks my editor indent/unindent commands. 

Hence, switching to just always using 2 spaces for indentation in Elm seems to be an easy win, but try as I might, I just cannot reconcile myself to using two spaces for indentation, it just doesn't feel enough. So for the time being I'm going to live with the inconsistency.


