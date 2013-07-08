---
layout: post
title: "Linux command line Tips: Become a master"
date: 2013-05-19 01:24
comments: true
author: cyrilf
categories:
- linux
- tips
---

You're using the command line from time to time and you want to go further? You're at the right place.  
You're going to learn new tricks, use them and improve your skills with the command line.

Ready? Go ahead.

<!-- more -->

### Move in linux

`cd`, for Change Directory allows you to move easily and quickly across directories in your system.

> Ok Linux. I want to go to my pictures directory

    cd ~/Pictures

> Ok, now I want to go back home

    cd

> Good, and if I want to move back a folder

    cd ..

> Nice. I want to return to the last directory I was before, possible?

    cd -

> Awesome! Thanks.

Don't forget to use the tab key `⇥` for auto-completion, it's very useful!

### History master

You type many commands in your terminal, 10, then 100, then 100 000 000 ...
You know that you've typed a super-shinny-powerful command in the past but you can't remember what it was. You're going to discover how to find it back.

+ Up Arrow Key 

    To find the previous commands you've typed, you can use the upper arrow key `↑` (multiple times) and it will show you the history. Press Enter `⏎` when you find what you want.

+ History

    A usefull command in linux is `history`, it allows you to see a list of commands you've done previously.
    You can combine it with grep to find something specific.

        history | grep "myarticle.txt"

    On the left of every commands listed in the history you can see a number. You can use it to execute a specific command.

        ![number]
        !951      # execute the command number 951

+ Ctrl + R

    But if you want to be quicker you have to use `Ctrl + R`.
    It will display "reverse-i-search" and you can type any combination of letters. For example, "myarticle".  
    This will display a match of the most recent command in your history containing "myarticle".
    When you find what you were looking for, press Enter `⏎` to execute the suggested command.

+ Exclamation point or "bang"

    + "bang bang" command

        > I've typed a command that requires privileges but I've forgot to use `sudo`.  
        Any advice?

        Enter `!!` (known as "bang bang") to run the previous command.

            apt-get update
            sudo !!         # will run sudo apt-get update

    + Exclamation point

        > Ok, I have a super-awesome memory and I can remember every commands I've typed. How can I run what I've typed seven commands ago ?

        Easy ! Use `!-[number]`

            !-7

        > I want to find the last command that started with a specific text (e.g. "ls")

        Use `![expression]`

            !ls             # will run ls -la ~/Pictures

        > I want to get the previous command without the first word

        Use `!*`

            cp article.txt ./blog/article.txt
            mv !*         # will run mv article.txt ./blog/article.txt

        > I want to get the first argument of the previous command

        Use `!^`

            cp article.txt ./blog/article.txt
            nano !^         # will run nano article.txt

        > I want to get the last argument of the previous command

        Use `!$`

            cp article.txt ./blog/article.txt
            nano !$         # will run nano ./blog/article.txt

        To go a little bit further you can use all the previous commands with the following syntax:

        `[command]:p` : It prints out the command instead of running it

            !!:p             # prints (but don't execute) nano my-so-long-file-name.txt

        > I've seen that the history keeps only my 1000 previous commands. How to (de|in)crease this limit ?

        You can change this limit in your `~/.bashrc`. Look at the line with `HISTSIZE=1000` and update it to fit your needs.

If you've written some bad or shameful things in your history you can simply erase it with

    history -c

### Play with jobs

When you execute a script or a command that can take a long time to execute, you can run it as a background job.

Run a process in background by appending an ampersand `&`

    node server.js &

> Wait, wait! I forgot to add the ampersand. I need to kill the process, right?

The answer is : No. You can put the current job to the background without having to kill it.

To do so, you need `Ctrl + Z` to suspend the current job.  
Then, `bg` to run it in background.

> Ok fine, my job is working in the background but how do I see it ?

Simple you can use,

    jobs

Or

    ps

It displays a list of the processes you've launched.

Now if you want to kill a specific job, just use `kill`.

If you've choosen to use `jobs`, you can kill a process by is job-number in the list.

    kill %[job-number]

    kill %2

If you've choosen to use `ps`, you can kill a process by is process id (PID).

    kill [PID]

    kill 1293

To kill the current process you can also use `Ctrl + C`

### Do two things (or more) at once

Use a semicolon `;` 

    cd ~/prog/linux; nano article.txt

It's really usefull in some case when you write scripts or when you have a GitHub repository.  
For example, you can do :

    git clone git@github.com:requiremind/requiremind.github.io.git requiremind ; cd requiremind

To go further, if you want the second instruction to be executed only if the first exited with no errors, then you should use a `&&`

    ./testSomething.sh && echo "You can read me because no previous error happened"

### Shortcuts

> I'm writing a command and suddenly I realize that this isn't what I wanted to type or I've made a typo in the first word. What do I do now?

So this is what you usually do (based on my case):

+ Press Shift key `⇧` and top key `↖`: you're in a terminal, this isn't working..
+ Press the erase key `⌫` to the left of the line: works, but this is boring..

No more wasted time ! Now you can use `Ctrl + U`.   
Simple, easy, perfect !

And if you want to just erase word by word, you can use `Ctrl + W`.

### Define some alias

Sometimes you're going to face a hard-bad-long-painful-forgettable command. Unless you love the pain to type over and over again the same command, you can use `alias`. The syntax to define one is:

    alias [alias-name]=[command]

If you want your alias to persist across different sessions (close the terminal, reboot, ..) you have to add it to your `~/.bashrc`. So open your bashrc file and look for the `#some more aliases` section. Below this, type all your aliases. For example:

    alias ai='sudo apt-get install'
    alias au='sudo apt-get update'

Save your file. And you're done! Time saved.

### The end

Ok, now you've seen a lot of useful command line tips. Learn them, use them and enjoy them. Take your time and once you're done, use `exit` to close the console and enjoy real life !

    exit

Later on this blog will come another article based on `zsh` and `oh-my-zsh` to go even further and be more effective.

If you're on Twitter and you want to follow funny accounts about Linux commands I recommend : 

+ [@climagic](http://twitter.com/climagic)
+ [@UnixToolTip](http://twitter.com/unixtooltip)

And don't forget to follow us too [@requiremind](http:///twitter.com/requiremind)