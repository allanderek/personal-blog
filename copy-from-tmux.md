---
title: "Copy from tmux"
tags: programming
date: 2021-04-09
---

This is mostly a post for myself to remind myself how to do this. I typically program on a remote server. I ssh in via the terminal and use tmux to manage sessions. I then edit code in neovim. This setup has a bunch of advantages and a bunch of disadvantages as well. One of the disadvantages is that I found it difficult to copy and paste from the tmux session in my terminal, to a completely different program running on my local machine, such as say my web-browser, or a chat client.

Today I came across [this blog post](https://ianthehenry.com/posts/tmux-psa/) explaining how to copy and paste from within tmux. Fortunately that also works for me, and copies it to my **local** machine's clipboard which is great because I can now use this to copy-and-paste lines of code elsewhere.

So, how to do this, without any additional configuration. First you enter 'navigation' mode in tmux, I'm told you can do this using `Alt-Space` but that is bound to something in my window manager, so you can also do `prefix-[`, where `prefix` is by default bound to `Ctrl-b` but almost everyone sets it to something else (`Ctrl-a` for me).

When in navigation mode you can use the normal vim keys to move up and down etc. Then, to start selecting you press `Space`, to copy the selection you press `Enter` and that's it. Selection is done in a block manner, but that can be quite useful, it's at least more useful than copying across splits.

One definitely slightly bad thing I still have is that it copies 'terminal' wise, therefore it doesn't know anything about your vim setup. For example I have visual whitespace which I find really helpful for seeing the (relative) line numbers of indented code. But that means when I copy I get the 'visual space' characters. So for example instead of `x = 10` I get `x␣=␣10`. Not great. However, I can instead just 'cat' or 'less' the file in the terminal directly and copy-and-paste from there, rather than from within Vim. 

Anyway I doubt this post is helpful to anyone but me, but there you have it.
