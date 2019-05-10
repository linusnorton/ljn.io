---
title: Node.js software architecture
date: 2019-05-03 09:00:00
lastmod: 2019-05-03 09:00:00
description: >-
  Blending the best of object orientated design into functional programming with node.js
layout: post
---

JavaScript was designed for the browser. What was originally small snippets of event driven UI code have evolved into full on pieces of software architecture like React and Angular.

===
One of the draws of node.js is that it opens up server side programming to front end developers, but the architectural patterns used in the browser are fundamentally different to a back end service. Combine a lot of developers with different backgrounds with JavaScript's immense flexibility and you'll soon realize that there is no common consensus on software architecture in the node.js world.
===

## Dependency management

One of the most common issues I see in the wider node.js community is a refusal to use any form of [Inversion of Control](https://en.wikipedia.org/wiki/Inversion_of_control). It's common to see dependencies imported directly into each function:

```javascript
const express = require("express");
const requestHandler = require("./a-file");
const config = require("./config");

const app = new express();
app.use(requestHandler);
app.listen(config.port);
```

This makes the code brittle and difficult to test. Using inversion of control it's possible to structure the code so that dependencies are passed in and can be mocked out while testing:

```javascript
function createService(app, requestHandler, port) {
  app.use(requestHandler);

  return () => app.listen(port);
}
```

*Note that a function is used instead of a class here. It could be either, concepts like Inversion of Control do not imply object orientated programming. From this point on I use classes*

Now this snippet is no longer responsible for obtaining it's own dependencies something needs to set it up. My preference is to use a dependency container:

```javascript
const express = require("express");
const requestHandler = require("./a-file");
const config = require("./config");

class Container {
  getService() {
    return new Service(express(), requestHandler, config.port);
  }
}

class Service {
  constructor(app, requestHandler, port) {
    this.app = app;
    this.requestHandler = requestHandler;
    this.port = port;
  }

  start() {
    this.app.use(this.requestHandler);
    this.app.listen(this.port);
  }
}

const container = new Container();

container.getService().start();
```

Granted this is more code but it would be split out into three files: one for the service, one for the container and one for the boot code.

There are full on dependency injection libraries for node.js but I try to adhere to the [principle of least power](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html) and try to stick to explicit wiring over annotation based injection. To each their own.

## Async / await

Factoring in asynchronous code can be an issue for the boot script as there is no top level await. If our requestHandler needs to load some reference data from a database the asynchronous call will propagate all the way back to the bootstrapping code:

```javascript
class Container {
  async getService() {
    const handler = await this.getRequestHandler();

    return new Service(express(), handler, config.port);
  }

  async getRequestHandler() {
    const repo = this.getSomeRepository();
    const data = await repo.getSomeData();

    return new RequestHandler(data);
  }
}

const container = new Container();

container.getService()
  .then(service => service.start())
  .catch(e => console.error(e));
```

Depending on the complexity of your set up this code may become slightly unwieldy. There are two approaches I've seen to bootstrapping asynchronous services:

```javascript
// IIFE
(async () => {
  try {
    const container = new Container();
    const service = await container.getService();

    service.start();
  }
  catch (err) {
    console.log(err);
  }
})();

// main function
async function main() {
  const container = new Container();
  const service = await container.getService();

  service.start();
}

main().catch(e => console.log(e));
```

## Domain Driven Design influence

I'm a big fan of [Domain Driven Design](https://en.wikipedia.org/wiki/Domain-driven_design) and loosely follow the layered architecture it suggests. I have a preference for [DDD folder structures](https://www.fabian-keller.de/blog/domain-driven-design-with-symfony-a-folder-structure) where code is grouped by domain rather than function:

```
├── src
│   ├── ...
│   ├── User
│   │   ├── Api <-- controllers, view models etc
│   │   ├── Domain <-- entities, services etc
│   │   ├── Infrastructure <-- repositories, external APIs
│   │   └── Tests
│   ├── Order
|   |   ├── ...
```

It's a minor detail but it makes it easier to navigate the project and extract modules if the project gets too large.

The minimalist in me usually stops short of adding the next level of folder until I need it. I also still tend to have a separate test folder at the root level but that's more habit than design choice.

## Functional influence

While my software architecture is influenced by DDD I would say I'm equally or more influenced by functional programming. As a result my entity classes tend to be data structures rather than models that have functionality. TypeScript has support for `readonly` properties and while they don't guarantee immutability they are useful:

```javascript
class User {
  constructor(
    public readonly name: string,
    public readonly email: string,
    public readonly age: string
  ) {}
}
```

As the class is immutable it does not need getters or setters, it's just a data structure. This is known as an [Anemic Domain Model](https://martinfowler.com/bliki/AnemicDomainModel.html), which is highly discouraged by Martin Fowler. My justification is that adding lots of methods into an entity is not following the single responsibility principle. I prefer to keep all logic and rules in the other areas of the domain: Cases, Services, Events etc.

```javascript
class UserRegistrationService {

  constructor(
    private readonly emailService: EmailService,
    private readonly registrationTemplate: Template
  ) {}

  register(user) {
    const content = this.registrationTemplate.render(user);

    this.emailService.send(content, user.email);
  }
}
```

When I do use classes they tend to only have a single public method (single responsibility), this makes the practical difference between using pure functions and classes minimal. It's not a hard rule, sometimes it's convenient to have more than one public method, it's just a pattern I've noticed in my code.

When writing back end services it quickly becomes apparent that shared mutable state is dangerous. Sharing resources like files and streams across multiple functions becomes a nightmare to debug as you don't know who changed what and when.

I'm a strong believer that mutable state itself is not evil, but it must be encapsulated and hide from public view. I've discussed [tactics for encapsulating mutable state](./2019-01-31-encapsulating-mutable-state) before. My hard

== MORE ==

## Principles

### Principles I don't find useful

### Principles I do find useful
