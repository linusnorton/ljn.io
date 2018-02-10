---
title: Parallel Node.js
date: 2017-01-29 21:00
lastmod: 2017-01-29 21:00
description: Overview of parallel processing libraries in node.js and their comparison with PHP and Scala.
layout: post
---

Recently I decided to port the script that generates transfer patterns from PHP to TypeScript (node.js). The motivation for that was mainly performance but also because I prefer the type safety of TypeScript. Node.js may seem like an odd choice for a CPU-bound application but it was a language I knew well and some preliminary tests I did showed it would half the time to generate transfer patterns. The PHP script takes 3 hours to generate a day’s worth of patterns on an 8 core processor so any improvement on that would be a blessing.

The other motivation was that some aspects of the transfer pattern generation are asynchronous, which is where node.js really excels. Naively I thought that node.js would also be good for parallel processing despite its inherently [single threaded nature](http://www.journaldev.com/7462/node-js-architecture-single-threaded-event-loop). 

At this point I should probably differentiate Parallel and Asynchronous. Parallel is many things happening in together at the same point in time. Typically this makes use of some form of distributed computing either on many cores on one machine or many cores over many machines.  Asynchronous on the other hand is non-sequential (or non-linear)  execution, something more more familiar for JavaScript programmers like making a HTTP request and continuing execution at another point in time.

PHP is also not typically used for CPU bound tasks because it’s raw performance is well behind compiled or even other interpreted languages. Despite that it is a well proven technology and has a good selection of libraries for parallel programming. My PHP implementation was using a nice library called [spork](https://github.com/kriswallsmith/spork). 

Spork is rather simple, but when it comes to parallel programming that’s a good thing. It’s a wrapper of the [PHP fork](http://php.net/manual/en/function.pcntl-fork.php) functions that makes good use of shared memory. The API is simple to use, pass it a collection and a function and it will call the function in parallel with each item in the collection.

When converting to the transfer pattern generator to node.js I had hoped to find an equivalent library. Node being node I knew there would be a lot of libraries to sift through but I was surprised at how hard it was to find a good one.

## Parallel.js

My first choice due to it’s popularity and well presented website (now removed). Unfortunately I quickly discovered that the project has largely been abandoned and does not work with node 6.6. I seem to have accidently spurred the project into life by commenting on [this issue](https://github.com/parallel-js/parallel.js/issues/123). At least some people have taken it upon themselves to spur the project back to life.

## webworker-threads

Another seemingly popular library that is actively maintained. This library also had the benefit of conforming to the new-ish [web worker specification](https://www.w3.org/TR/workers/). Unfortunately it had a rather debilitating inability to use require (or import) in the worker thread.

## threads
This library was slightly less popular but had a relatively long history compared to some of the other libraries. It also appears well maintained. I did successfully get an implementation working with this library but found that the shared memory was rather limited and incapable of dealing with large collections.

## EMS

My eventual choice was a library called [EMS](https://github.com/syntheticsemantics/ems). It’s also relatively new but seems to be well grounded in theory. Mostly importantly it supports shared memory and I was able to get my program working with minimal pain. The only downside is that there is still a lot of work to do on the library, things like a parallel map function is not yet implemented (though it’s listed as TODO in the documentation).

## Conclusion

The moral of the story is never assume something will be easy even in a language/ecosystem as popular as node.js. Fundamentally node is built to be single process, single thread. It’s just how it works and pushing it beyond that is difficult because it’s not where the core developers want it to go. That’s not to say some use cases haven’t been catered for - there has been cluster management for multiple processes sharing connection resources for a while but again shared memory seemed to be the issue.

After slogging away I did get a working implementation in node.js and the performance was better than PHP. There were issues though. I could never quite get the memory usage to be as low as I’d like and the build up of promises essentially acted like a memory leak.

I have I since re-written the algorithm in Scala which has parallel collections baked into the standard library. After a bit of pain (it’s a very imperative algorithm) it’s working very quickly. 
