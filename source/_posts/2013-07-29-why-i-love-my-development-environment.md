---
layout: post
title: "Why I love my development environment"
date: 2013-07-29 20:08
comments: true
author: Neil Rosenstech
categories: 
- linux
- tips
---

Today I've decided to talk about the tools I use on a day-to-day basis. Why? Because I love them. That's it, now you know that I'm not going to be objective. This being
said, if you don't know or havn't tried them yet, then maybe now is a good time to do so and make your own opinion about it.

<!-- more -->

Note: I'm using Ubuntu with gnome but most of the stuff I talk about is working on OS X too.

But enough talking... : p

### Z

z... yes this is its name. z is probably one of the most useful things I've discovered the past year. z is actually a script that remembers where you've `cd` for the past
x days/weeks/months (I don't know the threshold) and saves you a tremendous amount of time.

```
cd /home/neil/dev/prism_secret_project
```

Here I'm using the `cd` command as usual (well at least if you're not using zsh, but we'll get to that later) to enter my *prism_secret_project/* directory. Now that I've done
this once, z is aware of that directory. I can just type

```
z pris
```

for instance, hit `tab` and z will autocomplete my command:

```
z /home/neil/dev/prism_secret_project
```

At this point you just have to hit `Enter` to navigate to that directory. 

You probably get it by now, z is some sort of fuzzy matching tool for navigating on your os.

Here you will find the script and the installation instructions: [https://github.com/rupa/z](https://github.com/rupa/z).

### zsh with oh-my-zsh

zsh stands for Z Shell and replaces the default shell (bash). 
Let's see what we can do with it.

First you do not need to use `cd` anymore. Typing a directory name as a command and hitting `tab` or `Enter` is now enough to navigate.

```
/home/neil/dev/prism_secret_project
```

Just hit `Enter` and you'll be there. 
You don't have to hit `tab` to autocomplete each directory, you can just type:

```
/h/n/d/pr
# => hit tab here
/home/neil/dev/prism_secret_project
```

zsh will offer suggestions when you hit `tab` and when multiple directories/commands match what you've typed. What comes in handy here is that you can actually navigate through
these suggestions with the arrow keys.

When you make a typo, zsh will try and find the closest command. For instance if you type something like `lls` instead of `ls`...:

```
lls
zsh: correct 'lls' to 'ls' [nyae]?
```

Here you have four choices: no, yes, abort, edit. No will continue and execute the previously entered command. Yes will use the suggestion. Abort will just abort
everything (pretty much straightforward huh) and edit will allow you to edit your command if you wanted to type something similar.

With *oh-my-zsh* you'll be able to get a cool terminal theme and syntax highlighting. 
You'll also be able to see all the time the git branch you're currently using on your prompt:

```
-> requiremind.github.io git:(source)
```

*oh-my-zsh* brings with him a great number of plugins and really handy aliases.
For instance when I'm using git I always use these commands instead of the old ones:

+ `gst` for `git status`
+ `gp` for `git push`
+ `gl` for `git pull`
+ `gco` for `git checkout`

Some say that zsh can make coffee but I havn't tried yet...

You should not have any problem finding installation instructions for zsh, here is the page you'll be looking for if you think about using *oh-my-zsh*: 
[https://github.com/robbyrussell/oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh).

### Sublime Text

I won't go into details here, there are already plenty of blog posts that talk about Sublime Text and the whys and hows as to why it works wonders. So I'm just gonna
give you the main "shortcuts" I use that made me first like the editor and then love it.

+ you can actually `ctrl + click` on your document to place multiple cursors and edit multiple things the same way and at the same time
+ select some text and then hit `ctrl + d` multiple times: this will highlight all the occurencies of that text in the document allowing a quick edit again
+ use `ctrl + c` to copy the current line directly, `ctrl + v` to paste it above the line you're at, use `ctrl + x` to cut the line and `ctrl + l` to select it
+ use `ctrl + shift + up/down` to move the line
+ use `ctrl + p` to fuzzy open documents
+ use `ctrl + shift + p` then type *ssjs* and hit `Enter` to set the current highlighting to javascript for instance (*Set Syntax: JavaScript*)

And some more advantages:

+ you can set a layout with one to four columns and one to four rows
+ when you click on a file, you get to see a preview of that file without having to open it

That's it for today! : )

Don't hesitate to share your tips below!