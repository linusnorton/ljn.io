---
title: Async Stream Handlers
date: 2018-11-29 05:00
lastmod: 2018-11-29 05:00
description: >
  Mixing asynchronous code with a node.js stream can have unintended side-effects.
layout: post
---

All node.js streams have an in-built back pressure system to prevent data building up in memory if it cannot be processed quickly enough. Naively mixing asynchronous code into stream handling can disrupt that back pressure system and have unintended side-effects.

Under normal operation a stream will stop loading information from a source if the handler (or downward stream) is unable to keep up:

```javascript
const fs = require("fs");
const input = fs.createReadStream("in-file.txt", "utf8");

input.on("data", data => {
  performSlowFunction(data);
});
```

As `performSlowFunction` blocks the handler of the `data` event, the stream will stop loading data in from the source once the [highWaterMark](https://nodejs.org/api/stream.html#stream_buffering) is reached.

Mixing in an asynchronous operation such as inserting into a database will disrupt the back pressure mechanism:

```javascript
const fs = require("fs");
const input = fs.createReadStream("in-file.txt", "utf8");
let inserts = 0;

input.on("data", async (data) => {
  await insert(data);

  inserts++;
});
```

Under the hood, `async` functions are [generators that yield a promise](https://medium.com/siliconwat/how-javascript-async-await-works-3cab4b7d21da). In this case, as soon as the `insert` function generates a promise it's yielded back to stream handling code. The code that handles the stream back pressure is not designed to work with promises and all it sees is that the function returned, so it assumes it's safe to process more data.

It's possible that you could run this code and not encounter an issue. However, if you are reading a large file and the `insert` function is not processing data quickly enough you will find that memory usage quickly becomes an issue as more and more promises are left unresolved in memory.

There are two ways to safely mix asynchronous code and streams.

# Async iterators

Node.js 10+ comes with experimental support for [async iterators](https://thecodebarbarian.com/getting-started-with-async-iterators-in-node-js). If you are happy to use an experimental feature, they are a simple, idiomatic way of keeping back pressure while working with asynchronous streams:

```javascript
const fs = require("fs");
const input = fs.createReadStream("in-file.txt", "utf8");
let inserts = 0;

for async (const data of input) {
  await insert(data);

  inserts++;
};
```

From node.js 10, inbuilt node streams have been given a `Symbol.asyncIterator` property. Unlike traditional iterators, this iterator returns a promise of data instead of the data itself.

The data inside an async iterator can be accessed using a `for async` loop. In our case, this removes the need for the stream handler function and allows us to write the code inline.

The `for async` loop is syntactic sugar that awaits each item in the `input` stream, but it also has the advantage that any asynchronous code inside the for loop is completed before reaching for the next item in the stream.

There is a performance penalty to pay with async iterators as both the handling and the reading use promises. This performance penalty [has been reduced](https://v8.dev/blog/fast-async) but might be an issue depending on your use case.

# More streams

If you do not wish to use an experimental feature, the best way to safely process the asynchronous code is to encapsulate it inside a custom `Transform` or `Writable` stream so that the back pressure can be managed manually:

```javascript
const fs = require("fs");
const Writable = require("stream").Writable;
const input = fs.createReadStream("in-file.txt", "utf8");
let inserts = 0;

const insertStream = new Writable({
  decodeStrings: false,
  write: async (chunk, encoding, callback) => {
    try {
      await insert(data);
      inserts++;

      callback();      
    }
    catch (err) {
      callback(err);
    }    
  }
});

input.pipe(insertStream);
```

The key here is that that the `callback` function is not executed until the asynchronous code has been completed. It's also worth noting that the `try..catch` statement catches any errors and propagates them back to the input stream using the `callback` function.

This approach makes good use of the back pressure system by allowing the input stream to load more data while the asynchronous code is awaiting callback, but also respects the `highWaterMark` and prevents too much data building up in memory.

It is slightly more verbose but it may perform better and it will not give you a warning in the console about using experimental features.
