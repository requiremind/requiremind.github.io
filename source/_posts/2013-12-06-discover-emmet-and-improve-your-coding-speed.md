---
layout: post
title: "Discover Emmet and improve your coding speed"
date: 2013-12-06 23:23
comments: true
author: cyrilf
categories:
- tips
- code
---

If you're here, I assume you're a passionate web developer. If not, you'll become one after reading this article.
Indeed, [Emmet](http://emmet.io) is an essential toolkit for you! It will improve your coding speed instantly and easily!

You seem interested, then keep reading to discover more.

<!-- more -->

{% img center /images/emmet/fast-train-lights.jpg 'fast train' 'fast train' %}

### Emmet

Ok, to begin I want to say that if you already heard about Zen Coding, then you'll find that Emmet is pretty similar.
In fact, Zen Coding has evolved into Emmet adding a bunch of new cool features that you'll discover soon.

In a nutshell, Emmet is a toolkit for web developpers that improves your HTML and CSS workflow.
You'll use CSS-like expressions to generate your HTML and a smart syntax to generate your CSS.

Enough talking, as always the best is to see some examples!

### HTML

Warning! This is an important step!  
Once you'll read and learn what is following, you'll

  + First, fall in love with these new coding habits.
  + Second, you'll never, ever type a full tag anymore in your html.
  + Third you'll save an amazing amount of time.

So if you are ok with this, keep reading.

Let's start with the easiest example:

``` html 
<div></div>
```

can now be summarized as:

    div[tab]

_(when I use [tab] it means, press the tab key `⇥`)_  
[tab] is the default key for executing Emmet in your code editor. It will dynamically parse the expression and generate the final html.  
Here, it simply generates a `div` element.

> Ok, that's cool but is it all?

Of course not! Let's see some magic happening!

``` html
<ul>
  <li>
    <p id="1">Element 1</p>
  </li>
  <li>
    <p id="2">Element 2</p>
  </li>
  <li>
    <p id="3">Element 3</p>
  </li>
</ul>
```
can be simplified by using:

    ul>li*3>p#${Element $} [tab]

Ok let's review this one, step by step.

`ul`is our parent element.  
Then we use the `>` symbol which represents a child in html. `ul` contains `li`.  
The `*` is a multiplication operator that allows us to define how many times we want to output the element. Here 3 times.  
In each of these elements we got a `p` element with an id (represented by the `#`, for a class we should have used a `.`).  
The `$` is a specific symbol. It's an integer that will increment by one for each item.  
The curly braces `{}` represent the text. Here Element plus the current item number.

Ok I feel that you want one more example. Here you are:

``` html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8" />
    <title>Document</title>
</head>
<body>
    
</body>
</html>
```
is now as simple as:

    html:5 [tab]

I hope that now you understand the concept and mainly you see the opportunities.

As this article's goal is just to make you discover this tool, we won't go deeper with the html examples.  
You can refer to the [official documenation](http://docs.emmet.io) whenever you need it.

### CSS

Emmet also allows you to write CSS faster than ever before. I really use it a lot and it's easy to learn.  
It defines a lot of shorcuts for the CSS properties.

I can't show them all, but here is a list of common examples (remember to press the `tab` key to execute Emmet):

    w23        => width: 23px;
    w77p       => width: 77%;
    pos:a      => position: absolute;
    fr         => float: right;
    db         => display: block;
    tdu        => text-decoration: underline;
    f5#f       => font: 5px #fff
    bd+        => border:1px solid #000;

And it also takes care of the vendor prefixes:

    bdrs25 => -webkit-border-radius: 25px;
              -moz-border-radius: 25px;
              border-radius: 25px;

It's so intuitive that you'll learn them easily by practising and discovering them by yourself.  
But as always, here is the [documentation](http://docs.emmet.io/css-abbreviations).

### Fuzzy search

Emmet is really smart and implements a cool feature called [fuzzy search](http://docs.emmet.io/css-abbreviations/fuzzy-search).
This one is being used when we write an unknown abbreviation. Emmet will try to find the closest snippet definition and apply it.

For instance, 

    fl:r  => float: right;

But we can break the convention and use `flr` instead. Emmet will use the fuzzy search to match `fl:r` and apply it.

### Code editors

The power of Emmet also resides on the fact that it's available on a large amount of code editors as you can see on their official [download page](http://emmet.io/download).

{% img center /images/emmet/editors.png 350 350 'code editors' 'code editors' %}

### Conclusion

So, this was the introduction to Emmet. I hope you will use it a lot and see the improvement in your coding speed.
If you know other tools or anything like this one, feel free to leave a comment, I'm definitely interested!

Here is my last present. If you're the kind of long cheat-sheet lovers then [here is your hapiness](http://docs.emmet.io/cheat-sheet). You won't be disappointed ;)