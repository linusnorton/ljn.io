---
title: Refactor to Functional
date: 2013-09-19 10:00
lastmod: 2013-09-19 15:15
description: Practical examples of refactoring to functional programming
layout: post
---

This article helps procedural or object oriented programmers refactor the way they think to a more functional approach using practical examples in JavaScript.

We will cover the cornerstones of functional programming such as function purity, recursion, map, reduce, curry and compose.

## Why Functional?

Functional programming can be hard to wrap your head around. Yes, that’s right you’re not the only one. At it’s worst functional programming is dense and looks mathematical but written correctly it is clear, concise and easy to debug.

Functions are described as “pure” when they fulfil two conditions:
* They always return the same result given the same input
* They have no side effects

For a function to always return the same result it cannot make any external system calls and cannot maintain any state. Classic examples of state would be the `this` variable or other global variables. These are things your function can change, but more importantly things your function relies on that other processes might change.

Similarly functions that have no side effects must not mutate existing variables or perform calls to external systems (APIs, file system etc).

Why does this matter? Pure functions are incredibly easy to test, they take 0 or more inputs and return a value. This means no mocking of dependencies or setting test harness to get the system in the correct state. Just simple input / output.

At the heart of functional programming is a belief that data is just data and the program’s core responsibility is to transform and manipulate that data.

## Map

Let’s say we have an array of numbers and we want to get the square root of each of them. That’s pretty easy right?

{% highlight javascript %}
function sqrtAll (values) {
    for (var i in values) {
        values[i] = Math.sqrt(values[i]);
    }

    return values;
}
{% endhighlight %}

This function mutates the state of the values array and returns the result, this has the potential to cause side effects and may lead to problems if other parts of the application contain references to the same array. It’s safer to return a new set of results. Using map we can return a new array of sqrt’d values. Map applies a given function over an array.

{% highlight javascript %}
function sqrtAll (values) {
    return map(Math.sqrt, values);
}
{% endhighlight %}

Not only is this fewer lines of code, it is “pure” and produces no side effects, but we can still do better.

##Curry

Functional currying is a way of partially applying a function. If that doesn’t help, think of it as a way of creating a new function based off an existing function with some arguments already supplied. Take this add function:

{% highlight javascript %}
function add (x, y) {
    return x + y;
}
{% endhighlight %}

We can curry it to make a new function called add5:

{% highlight javascript %}
var add5 = curry(add, 5);
add5(2); // Gives 7
{% endhighlight %}

On the surface of it this may seem fairly pointless, but it’s actually very powerful. Take our earlier example of `sqrtAll`, we can now define that as:

{% highlight javascript %}
var sqrtAll = curry(map, Math.sqrt);
{% endhighlight %}

Here we are saying take the map function and preload it with the Math.sqrt so all that’s left to do is provide the array of values:

{% highlight javascript %}
sqrtAll([9,64]); // [3, 8]
{% endhighlight %}

## Recursion

The join function takes an array of strings and joins them together with a separator. Here is a typical implementation:

{% highlight javascript %}
function join (values, separator) {
    var string = values.shift();

    for (var i in values) {
        string += separator + values[i];
    }

    return string;
}
{% endhighlight %}

We can also implement this using recursion:

{% highlight javascript %}
function join (separator, values) {
    return values.length === 1
            ? values[0]
            : values[0] + separator + join(separator, values.slice(1));

}
{% endhighlight %}

This function deals with two scenarios: it has reached the end of this list, in which case it just returns the value or it is not at the end and returns the next value + separator + the result of join with the rest of the array. This function will keep calling itself removing one from the array each time until it reaches the end.

## Reduce

Reduce is a special case of recursion where a data structure (e.g. an array)  is reduced down to single value. For this example we will look at sum.

{% highlight javascript %}
function sum (values) {
    var total = 0;

    for (var i in values) {
        total += values[i];
    }

    return total;
}

var total = sum([1, 2, 3]); // 6
{% endhighlight %}

The procedural version of this code looks much the same as our join function. The reduce function handles our recursion so all we need to do is the addition.

{% highlight javascript %}
function add (value, accumulator) {
    return value + accumulator;
}

var total = reduce(add, [1, 2, 3]); // 6
{% endhighlight %}

Or when combined with curry:

{% highlight javascript %}
var sum = curry(reduce, add);
var total = sum([1, 2, 3]); // 6
{% endhighlight %}

In a normal reduce implementation it will be expecting 2 or 3 arguments; the function, the array and an optional accumulator (starting point). The function will be recursively called for each item in the array, each time receiving the value and current accumulated result.

## Compose

Using compose we can chain together any number of functions to a new function. This is great if you need to perform a number of actions on the same dataset. Let’s take our earlier examples of `sqrtAll` and `sum`. If we want to get the total of the sqrt of each item in an array we could write:

{% highlight javascript %}
var squareRoots = sqrtAll([9, 64]);
var total = sum(squareRoots);
{% endhighlight %}

But it is much more concise to use compose:

{% highlight javascript %}
var total = compose(sum, sqrtAll)([9, 64]);
{% endhighlight %}

This code reads very nicely, albeit from right to left. With this array, square root each item and sum the result.

## Real World Examples

All the examples so far have been fairly artificial to keep them short and to the point. Now let’s go over some more real world scenarios.

### jQuery Callbacks

jQuery ajax requests have callbacks for error and success handling. Quite often the we will want to provide some UI feedback based on the request result. Let’s say we want to display a dialog box with either an error message or a success message we could write it like this:

{% highlight javascript %}
// let’s start a loading icon
startLoadingSpinner();

$.get('/api/endpoint', {
    success: displaySuccess,
    error: displayError
});

function displaySuccess() {
    stopLoadingSpinner();
    showDialog('Success!');
}

function displayError() {
    stopLoadingSpinner();
    showDialog('Oh no, error!');
}
{% endhighlight %}

Obviously the `displayError` and `displaySuccess` functions are very similar and need to be refactored to remove the duplicate code. Now we’ll use curry to tidy things up a bit:

{% highlight javascript %}
startLoadingSpinner();

$.get('/api/endpoint', {
    success: curry(requestComplete, 'Success!'),
    error: curry(requestComplete, 'Oh no, error!')
});

function requestComplete(message) {
    stopLoadingSpinner();
    showDialog(message);
}
{% endhighlight %}

### Event Stream

In this scenario we have a list of events containing a type (create, edit, delete) and a date (yyyy-mm-dd). We want to roll up all the events of the same type on the same day into a single event.

We can make a composite key of the date and type to remove duplicates.

{% highlight javascript %}
function reduceEvents(events) {
    var eventStream = {};

    for (var i in events) {
        var event = events[i];
        eventStream[event.type + event.date] = event;
    }

    return events;
}
{% endhighlight %}

Pretty easy right? Now let’s add a couple of complications:
* The events are ordered by date descending (most recent first) and the current function keeps oldest event. We want to display the most recent event.
* We want to limit the final result to 10

Now we have:

{% highlight javascript %}
function reduceEvents (events) {
    var eventStream = {};

    events.reverse();

    for (var i in events) {
        var event = events[i];
        eventStream[event.type + event.date] = event;
    }

    events.reverse();

    return events.slice(0, 10);
}
{% endhighlight %}

This code is not complicated but to someone reading it for the first time it is not immediately obvious what is happening until they read the entire function. Using a few functional techniques we can make it more conscise and readable:

{% highlight javascript %}

var reduceEvent = curry(reduce, reduceEvent);
var first10 = curry(slice, 10);

var reduceEvents = compose(
    first10,
    reverse,
    reduceEvent,
    reverse
);

function reduceEvent (event, accumulator) {
    accumulator[event.type + event.date] = event;

    return accumulator;
}
{% endhighlight %}

What’s nice about this is that the reduceEvents function is much more descriptive. Rather than reading through the code we can just see what it is doing: with the events, do a reverse, reduce them, reverse them back and take the first 10. If you don’t like reading from right to left some libraries provide a chain function so it could be written as:

{% highlight javascript %}
return chain(events)
        .then(reverse)
        .then(reduceEvent)
        .then(reverse)
        .then(first10)
        .values();
{% endhighlight %}

Speaking of libraries the examples here will work with [fun-js](https://github.com/briansorahan/fun-js) or [scoreunder](https://github.com/loop-recur/scoreunder) (if you add \_. to the function calls). [Underscore](http://underscore.org) is a great library but the ordering of parameters makes it [difficult to curry and compose](http://www.youtube.com/watch?v=m3svKOdZijA).

## Conclusion

I hope this has helped highlight that functional programming can be clear and concise. Functions like curry and compose help you create a more human readible codebase so any one unfamiliar with the code can get up to speed in no time.

Functional progamming is definitely a different way of thinking and when I made the switch I didn't realise how engrained my procedural point of view was. Over time it becomes more and more natural to do things the functional way and I'm a better coder for it.

I'd like to catelogue some common functional refactors so if you come across any scenarios of your own please [get in touch](http://www.twitter.com/linusnorton).

# Contributions

Thanks go out to [Richard Edwards](https://github.com/apergy), [Perry Harlock](https://github.com/perryharlock), [Andrew Lawson](https://github.com/adlawson), [Rowan Manning](https://github.com/rowanmanning), [Simon Flack](https://github.com/simonflk), [Phil Booth](https://github.com/philbooth) for proof reading and pointing out mistakes.

