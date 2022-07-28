---
title: "Missing IDE feature"
tags: programming
date: 2021-03-05
---

I use neovim to edit code and I'm mostly editing Elm code, so the missing feature I'm about to describe is missing from neovim and for editing Elm code, I don't know of any other IDE that has this, but it wouldn't entirely surprise me if, for example, the Elm plugin for the IntelliJ IDE has this feature. I also think this feature would benefit most programming languages. Anyway, this feature regards editing the imports at the top of an Elm file. What I really want to be able to do is just press some key combination to bring up a dialog which allows me to edit the imports from where I am, without having to scroll up to the edits. What I mostly do now, is just edit code as if I had already edited the imports, press save, which tells me what imports I'm missing (and also if I scroll up there, which imports are no longer necessary). So I guess this is not a terribly needed feature, but I find myself wishing for it quite commonly.

I also tend do the following whilst editing some code:

```vim
gg " take me back to the top
} " takes me to the top of the import list
" Then edit the import list, which might involve pasting from a register or just typing.
2 Ctrl-o " Takes me back to where I was editing before.
```

This mostly satisfies, however it strikes me that many programming languages have a kind of *'top-matter'* to each file that is mostly editable entirely independently from the rest of the file. So I believe this missing feature would benefit many languages. If not you can take the above as a bit of a tip.

Another option is to do `/^import ` which will take you to any line which begins with the word 'import' (note there is a space after the 't' in the vim command. Then once you've edited you can just do `Ctrl-o` to go back to where you were.

The small disadvantage to both `Ctrl-o` solutions is that it fills up your list of locations. So again, I still think my missing feature has merit.

