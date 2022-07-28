---
title: "Domain Specific Languages and frequency"
tags: programming
date: 2021-02-26
---

When I started programming almost all programs were built using [make](https://www.gnu.org/software/make/), which is basically a domain specific language for writing down dependencies and compilation instructions. It seems to have mostly fallen out of fashion, I think largely because many languages take the hard part of figuring out what depends on what out of the hands of the programmer and just do the correct thing when you call the compiler against the single file which hosts the `main` (or equivalent) of the program. You could still use make files for things like documentation, but what people tend to do instead is just use a script, they don't worry too much about dependencies for things other than compilation, because you can generally just always run everything. 

My problem with `make` was that it was a completely separate language that one had to learn. That is definitely useful in some circumstances, the problem with make is that once you've set up how to build your program, you mostly don't really have to touch the make file again very frequently. Unless you're building a new program weekly, that means you aren't writing `make` very often. That means that every time you **do** you pretty much have to re-learn it all from scratch again. Even this was mostly fine, except when you have to do something slightly different.

Here is a standard sort of make file taken from the [gnu make documentation](https://www.gnu.org/software/make/manual/make.html):

```make
edit : main.o kbd.o command.o display.o \
       insert.o search.o files.o utils.o
        cc -o edit main.o kbd.o command.o display.o \
                   insert.o search.o files.o utils.o

main.o : main.c defs.h
        cc -c main.c
kbd.o : kbd.c defs.h command.h
        cc -c kbd.c
command.o : command.c defs.h command.h
        cc -c command.c
display.o : display.c defs.h buffer.h
        cc -c display.c
insert.o : insert.c defs.h buffer.h
        cc -c insert.c
search.o : search.c defs.h buffer.h
        cc -c search.c
files.o : files.c defs.h buffer.h command.h
        cc -c files.c
utils.o : utils.c defs.h
        cc -c utils.c
clean :
        rm edit main.o kbd.o command.o display.o \
           insert.o search.o files.o utils.o
```

As you can see the syntax is pretty reasonable (though it weirdly insists on the use of tabs). You can probably roughly understand this, for instance the following rule means:

```make
main.o : main.c defs.h
        cc -c main.c
```
To build the file `make.o` you have to run the command `cc -c main.c`, you should do this if either `main.c` or `defs.h` change. Because make files were awkward to keep up-to-date, you typically had a program that generated this, or at least a part of the make file, by analysing your program. It followed unix philosophy quite well, in that each program did one thing well, but as I said above nowadays most compiler writers just put the whole thing in one executable. The compiler, the dependency analyser, and the build tool. Anyway that's not really the point I wish to make today.

The point I wish to make today is that `make` is a not a general purpose program, but inevitably you'll start wanting to do some general purpose programming type things. For example, maybe you want to build the documentation, and rather than maintain a list of all the source files you just want a rule that looks like:

```make
docs : <all source files>
        docs-generator <all source files>
```
However, because `make` isn't a general purpose programming language, you cannot use your normal method of getting all files in a directory (in whatever programming language you're actually building the program in), and perhaps filtering them for those that are source files. I'm pretty sure there is some make magic that you can use to author the rule I'm proposing here, but two points:
1. Because you do not program in `make` on a daily basis you'll probabaly have to look it up, you'll be slower than if you were just writing your build script in either a scripting language or whatever your main language is
2. Okay maybe this particular thing is possible but sooner or later you're going to find something that isn't.

When you do come across something for which there is no `make` support, what you likely end up doing is either figuring out how `make` can call an external program to change the rules, or writing a script which builds the make file. This might be okay, as I said above, it's kind of the unix philosophy of doing one thing well. However, what probably then happens is that you realise your build can be wrong if you've not recently run your make-file-generator script, so now what you do is just **always** run the make-file-generator before running `make`. Now the make-file-generator could, instead of generating a make file to build the program, just go ahead and build the program.

Now there is an opposition to this idea. It's not that trivial to do the actual work of `make` which checks for each rule if the dependenes (eg. the `main.c` and `defs.h` in the rule I extracted above), have changed. Remember the dependency of one rule might be the target/output of another. So you have to do this transitively. My answer to this is sure, but if `make` was not a domain specific language, but instead a **library** then your make-file-generator, would indeed be just a build script, but one that uses the `make` library to do the common-but-non-trivial task of figuring out which targets need to be re-built based on the rules you have defined. The big difference is that now you're writing your rules in your normal general purpose programming language. This might look a bit uglier than how rules are defined in `make`, but **now** if you have to do something weird such as "just get all the source files in a directory", or "update all the copyright files in the project if this hasn't been done since the start of the year", you have the full expressivity of your general purpose programming language behind you. And, because you're generally programming in that language on a day-to-day basis, you're pretty proficient at basically any logic that the build script might need, unlike if you have to try to express that logic in `make` a DSL you dip into maybe 3/4 times a year. 



So what is my conclusion? I am certainly not claiming that domain specific languages have no place, they clearly do. Before you start building one though, ask yourself if a library might be a better option. I *think*, you will want to consider how often you expect your domain specific language to be used by its intended users. If the answer is *'pretty sparsely'*, then reconsider because what you're basically saying that your users will have to re-learn it everytime they use it. Another question is, can you really forsee all primitives that might be required in the domain specific language, or is it possible that someone will ask something like "how do I get the current date?" in your language. Lastly, can you see that after many feature requests you might end up with a poorly thought out general purpose programming language.

A final point here is that interpreted languages allow for doing the configuration of programs within the language the program is written in. Otherwise the configuration file is effectively a domain specific language. There are some ways in which this can also work for compiled languages, for example [elm-review](https://package.elm-lang.org/packages/jfmengels/elm-review/latest/) is configured in Elm itself. However this is always a bit more natural in an interpreted language.
