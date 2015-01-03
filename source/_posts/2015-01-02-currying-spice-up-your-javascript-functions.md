---
layout: post
title: "Currying, spice up your javascript functions"
date: 2015-01-02 14:41
comments: true
author: cyrilf
categories: 
- javascript
- learning
- learn-js
---

Currying is an interesting technique that will power up your functions usage. In this article we're going to explain it from basic example to some more advanced use cases and you'll see that knowing this pattern, will give you some extra power as a developer.

It's time to cook, let's spice it up!

<!-- more -->

{% img center /images/currying/spices.png 'spices' 'spices' %}

### Definition

To begin, a definition won't hurt.

So, currying allows you to invoke a function that build and return you another one prefilled with the arguments you gave.  You can see it as partially applying a function.
And now you're supposed to say:

> _What?!_

Yeah it might be blurry.. so it's a perfect time for a basic example!

### Basic example

``` js
var liker = function(type) {
	return function(name) {
		var result = 'I like this ' + type + ': ' + name;
		return result;
	};
};

var bookLiker = liker('book');
bookLiker('Oro');  // I like this book: Oro
bookLiker('Wild: A journey from lost to found'); // I like this book: Wild: A journey from lost to found

var tedLiker = liker('TED talk');
tedLiker('The art of misdirection'); // I like this TED talk: The art of misdirection

```

Ok so now it should be more clear for you. The `liker` is a _curried function_. As you can see, we use it to _prefill_ the first argument before the final function is executed.
This allows us to create more specific likers function as _bookLiker_ or _tedTalkLiker_ in our case.

> Cool, but.. what if we just specified two arguments in the first one?

Yes! You're right. The following function can easily do the same job:

``` js
var liker = function(type, name) {
	return 'I like this ' + type + ': ' + name;
}
```
But this one was a basic example to introduce you with this pattern.
It's now time to move forward into the concept.

### Advanced usage

We can write a curry helper function that is going to transform any "_standard_" function into a curried one.
We'll move from step to step on the implementation, so that you can absorb more easily the process.

So, let's start with the first one.

#### 1 - A naive implementation

``` js
function naiveCurry(fn) {
	var args = Array.prototype.slice.call(arguments, 1);
	return function() {
		return fn.apply(this, args.concat(
			Array.prototype.slice.call(arguments))
		);
	}
}
```

Let's have a look.
First, it stores the arguments passed to our function in a `args` property. Except for the first argument (the function we want to curry).
It then returns a function. When you then call this function, the old arguments (stored in the `args` property) are concatenated with the new arguments received and applied to the curried function `fn`.

_For those who didn't know, `arguments` is a specific word in Javascript and it corresponds to the arguments passed as parameters (even if they are not specified in the function signature). This `arguments` variable behaves like an `Array` but it's not from an `Array` type, that's why you're seeing those `Array.prototype` calls._

To illustrated it, a use case can be a `sendMessage` function:

``` js
var sendMessage = function(from, to, text) {
	return '@' + to + ': ' + text + ' - ' + from;
};

// return "@Irene Adler: Stop boring me and think. It's the new sexy - Sherlock Holmes"
sendMessage('Sherlock Holmes', 'Irene Adler', 'Stop boring me and think. It\'s the new sexy.');
naiveCurry(sendMessage, 'SH')('IA', 'Stop boring me and think. It\'s the new sexy.');
naiveCurry(sendMessage, 'SH', 'IA')('Stop boring me and think. It\'s the new sexy.');

var sendMessageFromSherlock = naiveCurry(sendMessage, 'SH');
sendMessageFromSherlock('IA', 'Stop boring me and think. It\'s the new sexy.');
```

Well that's fine, but as you can see the implementation is quite naive and can't resolve all the currying case, but we're moving towards the final syntax.

#### 2 - A better implementation

``` js
function curry(fn, length) {
    // Give us the function's arity (number of arguments)
    length = length || fn.length;
    return function () {
    	var allArgumentsSpecified = (arguments.length >= length);
    	// We have all arguments, we can apply them to the function
    	if(allArgumentsSpecified) {
    		return fn.apply(this, arguments);
    	}

        // We're missing some arguments, so we keep currying
        var partial = [fn].concat(Array.prototype.slice.call(arguments));
        return curry(naiveCurry.apply(this, partial), length - arguments.length);
    };
}
```

And the magic is on! This curry implementation is now way more robust.

``` js
var sendMessageCurried = curry(sendMessage);

sendMessageCurried('Sherlock', 'Watson', 'You see, but you do not observe');
sendMessageCurried('Sherlock')('Watson', 'You see, but you do not observe');
sendMessageCurried('Sherlock', 'Watson')('You see, but you do not observe');
sendMessageCurried('Sherlock')('Watson')('You see, but you do not observe');

var fromSherlocktoWatson = sendMessageCurried('Sherlock', 'Watson');
fromSherlocktoWatson('You see, but you do not observe');
```

Wohohoh! We've made it! And as you can see, it can be quite useful to have this syntax available. Furthermore, the curried function is totally transparent as it still can behave as a normal one (_look at the third line on this example, we can call it in a 'normal' way_).

And we're done! I hope you liked it.

> No. You can go further!

Wait, **what**? Isn't that enough? You're not impressed yet?  
Ok ok, it's only because I like you, nice reader, (and because you've managed to read it so far), that we're going to improve this function.

#### 3 - The spiciest curry function

I hope you're ready because this is going to be tastier than a [ghost pepper](http://en.wikipedia.org/wiki/Bhut_Jolokia) (a pepper 900.5 times hotter than a Tabasco sauce according to wikipedia!).

{% img center /images/currying/ghost-pepper.jpg 'ghost pepper' 'ghost pepper' %}

For this last example I'm using the `_` notation to refer to an 'empty' or a 'placeholder' variable.

``` js
var _ = {};
```
_Attention, this could enter in conflict if you're using a library as [lo-dash](https://lodash.com). So, to avoid this conflict you can easily replace the `_` variable on the following examples with the keyword `undefined`. Also, if you need one of your parameters to be equal to `{}`, this code won't work. So going with `undefined` is a way more bullet-proof solution. But the `_` notation is more simple to read._

``` js
function curry(fn, length, args, holes) {
    length = length || fn.length;
    args   = args   || [];
    holes  = holes  || [];

    return function() {
        var _args       = args.slice(),
            _holes      = holes.slice();

        // Store the length of the args and holes received
        var argLength   = _args.length,
            holeLength  = _holes.length;

        var allArgumentsSpecified = false;
        
        // Loop vars
        var arg     = null,
            i       = 0,
            aLength = arguments.length;

        for(; i < aLength; i++) {
            arg = arguments[i];

            if(arg === _ && holeLength) {
                holeLength--;
                _holes.push(_holes.shift()); // first hole became the last one
            } else if (arg === _) {
                _holes.push(argLength + i); // stores the hole's position
            } else if (holeLength) { // is there a hole available?
                holeLength--;
                _args.splice(_holes.shift(), 0, arg); // insert arg into the args list at the hole's index
            } else { // just an arg with no hole to fill, add him to the args list
                _args.push(arg);
            }
        }

        allArgumentsSpecified = (_args.length >= length);
        if(allArgumentsSpecified) {
        	return fn.apply(this, _args);
        }

        // keep currying
        return curry.call(this, fn, length, _args, _holes);
    };
}
```

This is it! The implementation is really different from our initial naive implementation. Because here we have to handle the `holes` and manage our `args` differently.
Basically, the main difference here is the `for` loop. We just keep references to our arguments `holes` and we use them to build our `args` correctly.
This is more complex but now the usages are really interesting.

Let's have a look:

``` js
// #1 - rgbCreator

var rgbCreator = curry(function(red, green, blue, alpha) {
	return 'rgba(' + red + ', ' + green + ', ' + blue + ', ' + alpha + ');';
});

// prints "rgba(204, 160, 29, 0.9);"
rgbCreator(204, 160, 29, .9);
rgbCreator(204)(160)(29)(.9);
rgbCreator(204, 160)(29)(.9);
rgbCreator(204, _, 29, .9)(160);

// prints "rgba(204, 160, 29, 0.5);"
var halfOpacity = rgbCreator(_, _, _, .5);
halfOpacity(204, 160, 29);

var shadeOfGrey = rgbCreator(0, 0, 0, _);
var opacity  = 0;
var shades   = [];
for(; opacity <= 1; opacity += 0.1) {
	shades.push(shadeOfGrey(opacity));
}

console.log(shades);

// #2 - bind functions
var logger = function() { console.log('Event occured!'); };

// We're expecting 3 arguments. It's due to the fact that document.addEventListener is a native function.
var bindEvent     = curry(document.addEventListener, 3)(_, _, false);
var bindClick     = bindEvent('click', _);
var bindMouseMove = bindEvent('mousemove', _);

bindClick(logger);
bindMouseMove(logger);

```

I kept the examples simple so that you easily understand the concept. If you run the second one in your console, you'll see that it's logging everytime you move/click with your mouse.


Voilà, you reached the end of this article! To go further you can even try to refactor and then add the `curry` function to the `Function.prototype` itself.  
I hope you've learn something new here and that you're going to use it on your own projects. Feel free to write any comments.  

If you want to discover more about Javascript, you can have a look to this [serie of articles](/categories/learn-js)
