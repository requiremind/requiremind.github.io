---
layout: post
title: "Memoization, speed up your javascript performance"
date: 2015-01-03 16:42
comments: true
author: cyrilf
categories:
- javascript
- learning
- learn-js
---

Now that web apps and websites are getting bigger and bigger, performances have become a really important point to focus on. This article will introduce you the concept of `memoization` that will allow you to improve your functions performances through some cache mechanisms.

Fasten your seatbelt, I'm starting the engine!

<!-- more -->

{% img center /images/memoize/turing.jpg 'turing' 'turing' %}

### Introduction

Memoization is a simple mechanism that consist in caching the result of computed values. This allows the next function call, done with the same parameters, to hit the cache rather than re-computing this values.  
The cache is indexed by the input arguments. If the arguments exist in the cache, then the cached value is returned. Otherwise, the function is executed and the newly computed value is added to the cache.

Let's see a basic example.

### Basic example

``` js
var speedAlert = function(speed) {
  var limitation = 75;
  // Imagine the following line being an extra CPU intensive task
  return speed > limitation;
};

var radar = {
  // some properties and functions
  // ...
};

radar.on('carDetected', function(car) {
  // The following function can then be called many times
  if(speedAlert(car.speed)) {
    radar.triggerFlash();
  }
});
```

As you can see the method `speedAlert` will be called many times. Imagine that inside this method there is a lot of computing going on (not just this basic operation :) ). So everytime we call it, we're going to use a lot of CPU time.

This is where `memoization` enter! Let's refactor the speedAlert method then.

``` js
var speedAlert = (function() {
  var cache      = {};
  var limitation = 75;

  function f(speed) {
    if(speed in cache) {
      console.log('hit cache');
      return cache[speed];
    }

    // Not in the cache, so compute the value and put it in the cache
    cache[speed] = speed > limitation;
    
    return cache[speed];
  }

  return f;
})();

// Now let's call this method manually
speedAlert(90); // return true
speedAlert(70); // return false
speedAlert(90); // hit cache + return true
```

In the last line of the previous example, you can see that the computation hasn't been made. The code has fetch the value from the cache and returned it.
This is the process of memoization.
When this method is called, it will look inside his cache to see if it had already compute the value for this `speed` parameter and return it, or compute the value and push it into the cache.

So now, everytime this method is called with the same argument, it won't compute nothing and this is a large amount of time earned.

I want to point out that here the `limitation` variable (75) is stored inside the function. This is an important fact to notice. Indeed, if this value was global or outside this function, we wouldn't be able to memoize `speedAlert` because then, his result would be influenced by some external properties and won't depends only on his inputs.

Here is a quick example of what I just said:

``` js
var limitation = 75;

var speedAlert = // same definition as before except the limitation assignation has been moved outside

speedAlert(70); // return false
speedAlert(70); // hit cache + return false
limitation = 50;
speedAlert(70); // hit cache + return false /!\ it should be true
```

You see, the result is already cached for the input `70`, so it doesn't care about the limitation change as it will never trigger the computation again.
It's important to understand that only the functions from which the result is influenced by the input are able to be memoized. This mean functions that always respect this: `f(x) === f(x)`

### Advanced usage

We saw memoization on a basic example with only a single argument. But this process can also handles multiple arguments. Plus, we were doing the `memoization` implementation inside the function but we can write a memoize function that will take a function as a parameter. This way, we can apply memoization to any functions.

``` js
function memoize(fn, resolver) {
  var memoized = function() {
    resolver  = resolver || JSON.stringify;

    var cache = memoized.cache;
    var args  = Array.prototype.slice.call(arguments);
    var key   = resolver.apply(this, args);

    if(key in cache) {
      console.log('hit cache');
      return cache[key];
    }

    var result = fn.apply(this, arguments);
    cache[key] = result;

    return result;

    // This could have been done with this one-liner too ;)
    // return (key in cache) ? cache[key] : (cache[key] = fn.apply(this, arguments));
  };

  memoized.cache = {};

  return memoized;
}

```

Time for explanations, the `memoize` function takes two parameters, a function (the one we want to memoize) and an _optional_ resolver.

Now that our `memoize` is more 'abstract', we can have multiple arguments passed to our function. So the `key` for the `cache` can't be as simple as the first argument anymore. That's why we're asking for a resolver. His goal is to take the `arguments` and compute a key for it. 

_Note: In the previous example, we're using `JSON.stringify` as a `resolver`, but this one is not a bullet-proof solution as it won't work with all cases, plus, keep in mind that `key in cache` syntax is definitely not optimzed but allows this example to stay clear and light.  
That's why the cache management here is really basic. A way to start optimizing it could be by using a [weak map](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) rather than an object (cool feature coming in ES6)_

Time has come to play with it!

``` js

var speedAlert = memoize(function(speed) {
  var limitation = 75;
  // Imagine the following line being an extra CPU intensive task
  return speed > limitation;
});

speedAlert(50); // return false
speedAlert(50); // hit cache + return false
speedAlert(90); // return true
speedAlert(90); // hit cache + return true
```

So now, our `speedAlert` method is a normal function declaration and the memoize wrapper handles all the memoization process for us. That is pretty neat.

That's it for this article I hope you..

> Don't stop here! How could I see that it improve my performance.

Ok ok, let's make a quick test with the well known [Fibonnaci sequence](http://en.wikipedia.org/wiki/Fibonacci_number). This function is really adapted to this example as it has a lot of recursive calls and regularly compute the same values.
We're going to use some nice functions called `console.time` and `console.timeEnd` that give us the ability to easily track the time in our console.

``` js
function fibonacci(n) {
  return (n === 0 || n === 1) ? n : fibonacci(n - 1) + fibonacci(n - 2);
}

var fibMemoize = memoize(fibonacci);
var iterations = 35;

console.time('non-memoized');
console.log(fibMemoize(iterations));
console.timeEnd('non-memoized'); // log 195.896ms

// Now this call is memoized, so it's just going to hit the cache
console.time('memoized');
console.log(fibMemoize(iterations));
console.timeEnd('memoized'); // log 0.193ms
```

And you're going to see that, the more you increment the `iterations` variable, the more the utility of the `memoize` function is important! _(but don't push it too far, I've managed to freeze Chrome with a 75 value..)_

So, for all your CPU intensive tasks, think about implementing some memoize mechanism.  
But don't forget some of the downsides. 

You can't use it on a function that doesn't return the same result for the same input. It's not adapted to fast executing functions or not often called ones. Plus, the cache management have to be improved for heavy usages with some notion of retention and so on.

Don't hesitate to share your opinion in the comments.

If you want to discover more about Javascript, you can have a look to this [serie of articles](/categories/learn-js)