---
title: "Elm errors with filenames"
tags: elm syntax 
date: 2021-01-16
---


Elm famously has excellent error messages, particularly for type errors. When the compiler detects errors in multiple files it outputs a little switch between which looks like this:

```
`This `NeverEditable` value is a:

    Field.Editable

But `withEditable` needs the 1st argument to be:

    Int

                                          Admin.Config.Orders.OrderTotals  ↑    
====o======================================================================o====
    ↓  Admin.Config.Basic


-- TYPE MISMATCH ------------------------------------ src/Admin/Config/Basic.elm

```

Although it's a little redundant to repeat the filename (technically it's a module name in the separator but since module names must line up to the filename it's still redundant), I find this pretty helpful.
Just one thing I would change, for the last error message, because you're not at that point switching between error messages you don't get a module or filename at the very bottom of the output:


```
-- TYPE MISMATCH ------------------------------------ src/Admin/Config/Basic.elm

The 1st argument to `withEditable` is not what I expect:

61|                 |> Field.withEditable Field.NeverEditable
                                          ^^^^^^^^^^^^^^^^^^^
This `NeverEditable` value is a:

    Field.Editable

But `withEditable` needs the 1st argument to be:

    Int
```

That's fine in this case, but Elm's error messages, whilst very helpful, are also somewhat verbose, and certainly can scroll off the page, even if the error is pretty simple. For example you often get error messages involving large record types which are printed over as many lines as there are fields in the record type. Basically though, what this means is that if you, for example have a watcher which re-compiles your Elm project in a terminal next to your editor, then if you save your editor and get a large error message, which isn't necessarily in the file you've just saved, then you end up having to scroll up in the terminal to see which file the error is in. It would be super helpful if at the bottom of the  last error the filename/module was repeated.

P.S. This is also true of elm-review which uses the same trick.
