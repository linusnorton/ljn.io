---
title: Notes on node.js streams
date: 2018-09-05 06:00
lastmod: 2018-09-05 06:00
description: Notes on using streams in node.js
layout: post
---

There's no doubt that streams are an essential part of the node.js design philosophy. Everything from stdin and stdout to file access is a stream. You may be using a library to abstract it into a more accessible form such as a promise, but underneath the hood it's still streams.

When it comes to handling large amounts of data, streams are not only faster, but they're often necessary. There are times when it is just not possible to fit everything into memory, so it must be streamed chunk by chunk.

Natively, node.js comes with three types of stream: `Readable`, `Writable`, and `Transform`. Streams are Event Emitters so the easiest way to use them is to hook into their events:

```
const fs = require("fs");
const input = fs.createReadStream("in-file.txt", "utf8");

input.on("data", data => console.log(data));
```

For many cases it's beneficial to connect streams together with pipes:

```
const fs = require("fs");
const input = fs.createReadStream("in-file.txt");

input.pipe(process.stdout);
```

## Piping to multiple destinations

One of the first questions I had when picking up streams was whether it is possible to pipe a single source into two destinations. The answer is a definite yes. As streams are just a form of Event Emitter, every stream reading the data is just another listener to the `data` event.

However, there is one difference between a stream and a standard Event Emitter: back pressure. When data is emitted from a `Readable` stream the listeners must declare that they have processed the data and are ready to receive more. This is a very useful feature as it prevents input streams clobbering output streams with too much information, but it does mean that you need to ensure all your listeners are processing the data. Take this example:

```
const fs = require("fs");
const Writable = require("stream").Writable;
const input = fs.createReadStream("in-file.txt");
const fileOutput = fs.createWriteStream("out-file.txt");
const consoleOutput = new Writable({
  write: (chunk, encoding, callback) => {
    console.log(chunk);
    callback();
  }
});

input.pipe(fileOutput);
input.pipe(consoleOutput);
```

Under normal circumstances this code will copy `in-file.txt` to `out-file.txt` and output it to the terminal. If the `consoleOutput` stream did not execute the `callback()` the input would build up and both output streams would eventually stop receiving data. Streams have an option, call the `highWaterMark` that determines how much information is stored in memory before blocking the stream.

## Why is my chunk a Buffer?

If you ran the code above, you might notice that it doesn't quite do what you would expect. The `out-file.txt` is created with the content of `in-file.txt`, but the console output looks something like `<Buffer 64 65 72 70 0a ...>`.

By default all stream chunks are a Buffer. If you specify an encoding type on the input stream then will arrive as strings:

```
const fs = require("fs");
const input = fs.createReadStream("in-file.txt", "utf8");

input.on("data", chunk => console.log(chunk));
```

But this is not enough to cover all cases. When implementing a `Writable` the chunk will always be a Buffer unless the `decodeStrings` option is set to false, or the stream is operating in `objectMode`.

```
const fs = require("fs");
const Writable = require("stream").Writable;
const input = fs.createReadStream("in-file.txt", "utf8");
const fileOutput = fs.createWriteStream("out-file.txt");
const consoleOutput = new Writable({
  decodeStrings: false,
  write: (chunk, encoding, callback) => {
    console.log(chunk);
    callback();
  }
});

input.pipe(fileOutput);
input.pipe(consoleOutput);
```

## Transforming Streams

A common pattern with streams is to load data from a `Readable` source, process it and and then output it to a `Writable` destination. A `Transform` stream is the intermediary processing step that sits between input and output. It is both a `Readable` and `Writable` stream.

```
const fs = require("fs");
const Transform = require("stream").Transform;
const input = fs.createReadStream("in-file.txt", "utf8");
const toUpper = new Transform({
  decodeStrings: false,
  transform: (chunk, encoding, callback) => {
    callback(null, chunk.toUpperCase());
  }
});

input
  .pipe(toUpper)
  .pipe(process.stdout);
```

Notice that a `Transform` stream is a type of `Writable` stream so it requires the `decodeStrings` or `objectMode` option.

If we want to do something more useful that convert text to upper case it is likely we will need to use `objectMode`.

```
const fs = require("fs");
const Transform = require("stream").Transform;
const input = fs.createReadStream("in-file.txt", "utf8");
const toCase = new Transform({
  objectMode: true,
  transform: (chunk, encoding, callback) => {
    callback(null, {
      lower: chunk.toLowerCase(),
      upper: chunk.toUpperCase()
    });
  }
});
const toUpper = new Transform({
  objectMode: true,
  transform: (chunk, encoding, callback) => {
    callback(null, chunk.upper);
  }
});

input
  .pipe(toCase)
  .pipe(toUpper)
  .pipe(process.stdout);
```

Note that any stream sending or receiving an object must be in `objectMode`.

## Filtering streams

It's possible to filter streams using a `Transform`, but it's important to remember that the callback must still be called for every chunk received. If the callback is not triggered the `Readable` source will not know data has been processed and may stop sending data.

If the chunk should be removed from the stream trigger the callback with no arguments:

```
const fs = require("fs");
const Transform = require("stream").Transform;
const input = fs.createReadStream("in-file.txt", "utf8");
const containsSpaces = new Transform({
  decodeStrings: false,
  transform: (chunk, encoding, callback) => {
    if (chunk.includes(" ")) {
      callback(null, chunk);      
    }
    else {
      callback();
    }
  }
});

input
  .pipe(containsSpaces)
  .pipe(process.stdout);
```

## Use the streams

Streams are powerful and streams are performant. They may not always be the easiest to use but if you care about performance I would always recommend them.
