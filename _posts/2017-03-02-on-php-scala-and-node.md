---
title: On PHP, Scala and node.js
date: 2017-03-02 08:00
lastmod: 2017-03-02 08:15
description: My thoughts and observations on PHP, Scala and node.js - three good but very different languages.
layout: post
---


Over the last few years I've been skitting round between PHP, Scala and node.js. All three are now mainstream languages with their pros and cons but more and more I find myself be most productive in node.js with the help of TypeScript before I examine why that is let's take at look at my experiences with PHP and Scala.

# PHP

PHP certainly has it's detractors, but to be honest I've always felt the criticism was slightly superficial and a bit overblown. Sometimes the tech community is like a playground and it's very easy to pick on the ugly kid. PHP is that kid. At [Assertis](https://www.assertis.co.uk) we run a team of about 12 developers with about 8 of those writing PHP code and I can tell you first hand they can deliver more than a 12 person Scala team.  There is no disputing PHP is a productive language.

Unfortunately PHP does have it's limits, the single page request model is the both a blessing and a curse. On the one hand you you don't have to worry about memory leaks (as much), connection pools and concurrency. On the other hand you can't pre-cache things in memory or take advantage connection pools or make use of parallel processing. 

Performance is an issue. Maybe not for your standard CRUD application but for anything more, it is. Even with the huge leap forward of PHP7 it still lags behind most other languages in raw computational power. Technically you can do parallel processing with it, but only via the CLI, not as part of the apache or fast-cgi module.

There's a very fundamental and deliberate trade off there - simplicity over power and performance. In my opinion people put too much stock in the latter two and not the former. 

That being said, PHP is not a joy to code in. Most modern languages now have better support for low level functional programming like shorthand anoymous functions, currying or partial application and other things. That's not to say you can't do functional programming, some of the tools you need are there (array_map, array_filter) it's just that the result is usually so syntactically ugly it's off putting and not really the "PHP way".

# Scala

I don't think you could pick a language more at the other end of the spectrum to PHP than Scala. It is one of the, if not the most conceptually advanced languages out there. The type system alone sets it apart but with the addition of pattern matching, implicits, packages and objects it really is a powerful system to work in. And it's fun... most of the time. As a technical guy I like to solve technical problems and Scala allows you to come up with some really elegant solutions. The issue is that every developer writes a Scala program in an entirely different way and too often it's very difficult to grasp other people's code. 

Libraries are another contentious issue in the Scala world. Too often using a library involves learning a new DSL that looks beautify but adds conceptual complexity to newcomers. The tendency to use symbols for everything certainly doesn't help people. It's not just the DSLs, when developers have all that power at their fingertips they tend to use it so it is hard to use Scala without very quickly coming up against monads, type classes and other complex concepts. There is a tendency for people who understand those things to forget that not everyone else knows them yet, being developer is a constant state of learning. 

Recently the Scala community is pushing more and more to the [principle of least power](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html) - do what you need to do using the least complex set of features you can. It's something I agree with wholeheartedly. I've always said that I strive to write simple code and when I speak to other developers they usually say the same. Calling someone's code complex is a quite an insult. Yet, simplicity means different things to different people. *It's about where you put the complexity*. By using advanced features like type classes you can make code more elegant and reusable, by using monads you are removing edge cases, concurrency issues and other difficult problems but you are adding conceptual complexity.

Simplicity is hard. Simple to read, simple to mentally model and simple to change is hard^3.

I still find Scala one of the most enjoyable languages to code in for business logic. Easy access to parallel processing, pattern matching and the collections library are just fun, and it's fast! Unfortunately, this enjoyment quickly fades when it comes to using libraries for HTTP servers, JSON marshalling/unmarshalling, database access or dealing with implicits that have "diverged". What's worse is when I look back at what I've delivered, part of me knows if I'd used a different language I would have been further along with the project. I learn a lot and become a better developer and that makes me feel good but it's quite a selfish trade-off to make on someone else's dime. 

# Node.js

The evolution of JavaScript over the last 10 years has been remarkable. After years of stagnation it's sprung to life with new features, better performance and a vibrant community. I would say there are three main reasons for this: higher expectations for website interfaces, node.js and coffeescript. Coffeescript is the one that people probably give the least credit to but if [Jeremy Ashkeas](https://en.wikipedia.org/wiki/Jeremy_Ashkenas) hadn't started adding new features by compiling coffeescript to JavaScript I don't think browser vendors would have realised it is in fact possible to add new features to JavaScript.

I have had an on/off relationship with JavaScript and node.js over the years. I've been through writing plain JS, to jQuery, to Backbone, to React, and while things have definitely improved I can certainly say I do not enjoy writing user interfaces.

When I began working with node.js. it was not great. It would frequently crash due to poor error handling on my part and in some cases the libraries. The [pyramid of doom](http://callbackhell.com/) was the thing that finally put me off. No matter how you broke down your code it just didn't read in a linear fashion. So I left for a bit.

Then came promises, they were an improvement so I came back and wrote an isomorphic app. It was fun, but dealing with nested arrays of promises was not. So I left.

Then came async/await. Now I can write the code I want to write. I can reason about asynchronous code without having to restructure my application.

In terms of actually using node.js, it performs well. V8 is being optimized all the time and now node.js is actually updating it things are improving. Node is still the king of low latency i/o bound tasks. Unfortunately, while it is designed for asynchronous code it is fundamentally it is inherently single threaded. You can fork processes and do things in parallel but support is limited and shared memory is hard. At the moment the most common advice if you need to do something like that you are using the wrong language as it is not great at CPU bound tasks either. 

I can't talk about node.js without mentioning libraries. They are a problem for JavaScript in general, not just node.js but they are a problem. Everyone and their dog has a library. On the one hand this is great - you can always find something you need - the problem is that it leads to a lot of very, very small libraries. If there are 5 libraries to something you can guarantee all 5 of those libraries will be in your dependency tree as the libraries you depend will all have chosen a different one. If you are working on the frontend it's a huge problem, you need to build those in to your package send your bloated 5mb .min.js file over a 3G network to a dated mobile phone.

Needless to say the quality of the libraries varies too.

I would say that with experience you learn how to pick out well supported and well written libraries but I think the community is maturing and now more weight is being thrown behind fewer libraries. 

# TypeScript

For me TypeScript was the final piece of the puzzle for node.js. It is essentially some parts of ES6/7/201x plus a type system. From ESx you get:

 - Classes
 - Decorators
 - Array destructuring
 - Async/await
 - Optional parameters

And TypeScript gives you:

 - Interfaces, abstract methods
 - Generics
 - Type inference
 - Type aliasing

Type systems are a much debated topic. Personally, I like them, provided they stay out of my way. That's one of the issues I have with Scala is that you eventually putting more and more logic and information into the type system. That's a very safe thing to do as your compiler can verify it is correct, but it's not something I find natural.

TypeScript walks the line between being correct and being practical very well. It is designed to integrate with other JavaScript code that has no type information so allows you gracefully deal with that. You can add type information to JavaScript code using the type definitions available via @types. When those definitions are out of date or incorrect you can manually patch them in your code until they are updated, or just not use the type information.

My favourite feature of Scala is pattern matching. I am also a big fan of the Option/Maybe monad, luckily there is a little known [TypeScript library](https://github.com/shogogg/ts-option) that provides these two things. The pattern matching is very limited but it is enough to be useful.

Type aliasing is another pragmatic feature. It's not as robust as Scala's value classes but it is "good enough". You can give a type like number and alias such as price Price and then everywhere you are using a price field it can have a type of Price. Handy if you need to refactor that into a class at a later point.

There aren't many downsides to TypeScript. Missing type declarations can be a pain, but a minor one. Compilation can be if you are working on the frontend. For node, just use [ts-node](https://github.com/TypeStrong/ts-node). Some would say it being developed by Microsoft is a bad thing, it made me skeptical at first, but I can say they are doing a much better job of running it as a community driven open source project than Facebook or Google do with their pet projects.

# Closing thoughts

Programmers like extremes. We take an idea like simplicity or power and see how far we can take it in search of more productivity. However, to me node.js doesn't feel like an extreme. It's not trying to invent new models of programming, it's not trying to dumb down your code to make it work no matter what, it's giving you a reasonable set of tools to use to do the job. It's trying to be accessible by taking the most widely used language and pushing it to the server. It has become [boring](http://mcfunley.com/choose-boring-technology), and that is a good thing.

If I were to have a scoring system it would probably be enjoyment to code in, how much do I deliver and how robust is the code I write. PHP scores well in delivery, Scala scores well in robustness, node.js scores well in enjoyment, well in delivery and to a lesser degree than Scala, well in robustness. 

Does this mean I will only write node.js code now? No, not at all. My job will still require I write PHP code from time to time. Performance will require that I still write Scala (or some other language) from time to time. But definitely for just getting stuff done on a day to day basis my first port of call is node.js.

