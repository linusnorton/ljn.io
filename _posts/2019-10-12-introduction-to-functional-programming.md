---
title: Introduction to functional programming
date: 2019-09-06 08:00:00
lastmod: 2019-09-06 08:00:00
description: >-
  A high level introduction to functional programming.
layout: post
---

This blog post serves as an accompanying article to the [introduction to functional programming]() talk given as part of Solirius' internal training. {: .article-headline}

# What makes programming functional?

Functional programming is often thought of as an alternative to object oriented programming, but this is not strictly true. Functional programming is an alternative to [imperative programming](https://en.wikipedia.org/wiki/Imperative_programming) that aims to be declarative and more grounded in mathematics.

An imperative program will modify the program state in the a computers memory to achieve the desired outcome. A functional program will apply a series of functions that transform data to the correct output. For example:

```javascript
function countGreaterThan10(numbers) {
  let total = 0;

  for (const num of numbers) {
    if (num > 10) {
      total++;
    }
  }

  return total;
}
```

A more functional approach would be:

```javascript
function countGreaterThan10(numbers) {
  return numbers.filter(n => n > 10).length;
}
```

Not only is this approach a lot less lines of code but it's more declarative and arguably easier to read.

# Higher order functions

One of the tenants of functional programming is that a function is just a type of data and so like any other data it can be passed to another function as an argument. When an function is passed to another function  it's often referred to as a higher order function.

The functional example of `countGreaterThan10` makes use of a higher order function to filter the array before returning it's length. 

Higher order functional are a common component of functional programming and if you've been using JavaScript it is likely you have already come across them. They are less common in stricter object orientated languages such as Java.

Another common example is the `map` function. In order to get the square root of every number in an array an imperative program would:

```javascript
function sqrtAll(values) {
  const result = [];
  
  for (const value of values) {
    result.push(Math.sqrt(value));
  }

  return result;
}
```

It's possible to achieve the same result using the `map` function. The `map` function applies a given function to every item an array and returns a new array with the result of each operation.

```javascript
function sqrtAll(values) {
    return values.map(Math.sqrt);
}
```

# Category theory and composition

Pipeline operator.


# Purity

One thing that is often mentioned as part of functional programming is 

## Mutability

# Types and monads
