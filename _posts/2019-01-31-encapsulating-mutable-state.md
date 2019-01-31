---
title: Encapsulating mutable state
date: 2019-01-31 05:00
lastmod: 2019-01-31 05:00
description: >
  Shared mutable state can make for an unpredictable program, this post covers a few strategies for avoiding it.
layout: post
---

If you've been bitten by the functional programming bug you may have heard that mutable state is [bad](https://hackernoon.com/mutability-leads-to-suffering-23671a0def6a), and should be avoided at all costs. Unfortunately, third party libraries are rife with mutable code and sometimes it's just very *convenient* to have mutable code.

So if mutable state can't be avoided, what can be done?

## Dangers of mutable state

On some level, all programs are mutable. Things have to change for a program to do something. Yet, one of the core principles of functional programming is to avoid shared mutable state. From a dogmatic perspective the argument is that you have to avoid shared mutable state in order to preserve [referential transparency](https://en.wikipedia.org/wiki/Referential_transparency) and bring the program closer to a mathematical foundation.

From a more practical standpoint, shared mutable state is often difficult to reason about and the source of many bugs.

For example, if an object is passed into a function and that function changes the object properties, it's hard to know what has happened without delving into the code to find out how it works.

It also means that the state of the program will change over time. If you call some code that changes some internal properties, then next time you call it the behaviour might be different.

In order to replicate a bug you have to replicate the entire state of the application at a given point in time. The more shared state there is the harder it is to replicate the state.

Generally, **local** or non-shared mutable state is not considered bad. If the state is encapsulated into a "local" scope, and not exposed to the outside then it cannot impact the behaviour of the overall system.

This example highlights some of the issues that can arise from exposing mutable state:

```javascript
class DocumentMerger {
  baseDocument = new Document();
  tableOfContents = new Page();
  lastPage = 1;

  constructor() {
    this.tableOfContents.addText("Table of contents");
    this.tableOfContents.addText("-----------------");
    this.baseDocument.appendPage(page);
  }

  merge(documents) {
    for (const document of documents) {
      // add the document to the end of the base document
      this.baseDocument.appendDocument(document);
      // add some text to the table of contents with the page number of the document
      this.tableOfContents.addText(document.getTitle() + "page " + this.lastPage);
      // update the total number of pages we've processed
      this.lastPage += document.getNumberOfPages();
    }

    // return the baseDocument with all the documents merged in
    return this.baseDocument;
  }
}
```

Assume that `Document` and `Page` are from a third party library that's been written in a mutable manner.

The `merge` method looks like it is immutable to the caller as it takes a list of documents and returns a single document, but there are a couple of problems in this example.

The first problem is that there is state left over after the method has been completed. If the method is called again the first set of documents will still be attached to the base document.

The second issue is that it is not **thread safe**. It is possible to have two threads enter the `merge` method at the same time and start appending documents to the base document. Not only does this mean the resulting document will contain both sets of documents in an undetermined order but it is likely that the page numbering will go awry too.

Multiple entry problems are rare in JavaScript as the main event loop is single threaded, but with the addition of `async/await` they are possible. In all threaded languages (Java, Scala, et al) thread safety common problem.

The root cause of both issues is that the mutable state is shared with the outside world. The solution is to **encapsulate the mutable state** by turning it into local mutable state. This will make the code safer, more transparent and easier to reason about.

## Move the state into a single function

The easiest, and often best, way to encapsulate state is to move it into a single function. Every time a function is called it gets it's own execution context and it's own set of local variables that are not shared:

```javascript
class DocumentMerger {

  merge(documents) {
    const baseDocument = new Document();
    const tableOfContents = new Page();
    let lastPage = 1;

    tableOfContents.addText("Table of contents");
    tableOfContents.addText("-----------------");
    baseDocument.appendPage(tableOfContents);

    for (const document of documents) {
      baseDocument.appendDocument(document);
      tableOfContents.add(document.getTitle() + "page " + lastPage);
      lastPage += document.getNumberOfPages();
    }

    return baseDocument;
  }

}
```

While this approach is often the natural choice it's not always possible. With more complex code this approach can often result in very large functions that are hard to follow.

## Nested functions (closures)

Expanding on the previous approach, it is possible to have functions nested inside the `merge` method that share the local scope. Technically speaking, these are [closures as opposed to lambdas](https://stackoverflow.com/a/220728):

```javascript
class DocumentMerger {

  merge(documents) {
    const baseDocument = new Document();
    const tableOfContents = new Page();
    let lastPage = 1;

    function initTableOfContents() {
      tableOfContents.addText("Table of contents");
      tableOfContents.addText("-----------------");
      baseDocument.appendPage(tableOfContents);
    }

    function addDocuments() {
      for (const document of documents) {
        baseDocument.appendDocument(document);
        tableOfContents.add(document.getTitle() + "page " + lastPage);
        lastPage += document.getNumberOfPages();
      }      
    }

    initTableOfContents();
    addDocuments();

    return baseDocument;
  }

}
```

This one is often down to personal preference and choice of language. In Scala this type of code is quite common and feels idiomatic, in Java it does not. As usual, JavaScript is a halfway-house in the wild west where some people will do this and others will not.

## Language features

Java and many other languages often have an in-built way of solving this. Using Java as an example, the `synchronized` keyword can be added to a method to ensure that there is only ever a single thread in the method at one time:

```java
class DocumentMerger {
  private Document baseDocument = new Document();
  private Page tableOfContents = new Page();
  private int lastPage = 1;

  DocumentMerger() {
    this.tableOfContents.addText("Table of contents");
    this.tableOfContents.addText("-----------------");
    this.baseDocument.appendPage(page);
  }

  synchronized public Document merge(List<Document> documents) {
    for (Document document : documents) {
      this.baseDocument.appendDocument(document);
      this.tableOfContents.add(document.getTitle() + "page " + this.lastPage);
      this.lastPage += document.getNumberOfPages();
    }

    return this.baseDocument;
  }

}
```

Unfortunately this only solves one issue as there is still state left behind after the method is executed. It should also be noted that there is also a performance penalty. The `synchronized` keyword will use a locking mechanism that blocks other threads, making this part of the code single threaded.

## Inner classes

Another approach would be to document that the class is stateful and that a new instance must be constructed every time `merge` is called. This is a flawed but unfortunately common approach. In my opinion it's naive to rely on developers on reading documentation, and it's bad practice to provide write code that can do something it should not do.

A better approach is to enforce good use by hiding the mutable state in an inner class and controlling the way it's called in the outer class:

```java
class DocumentMerger {

  public Document merge(List<Document> documents) {
    MutableDocumentMerger merger = new MutableDocumentMerger();

    return merger.merge(documents);
  }

  private class MutableDocumentMerger {
    private Document baseDocument = new Document();
    private Page tableOfContents = new Page();
    private int lastPage = 1;

    public MutableDocumentMerger() {
      this.tableOfContents.addText("Table of contents");
      this.tableOfContents.addText("-----------------");
      this.baseDocument.appendPage(page);
    }

    public Document merge(List<Document> documents) {
      for (Document document : documents) {
        this.baseDocument.appendDocument(document);
        this.tableOfContents.add(document.getTitle() + "page " + this.lastPage);
        this.lastPage += document.getNumberOfPages();
      }

      return this.baseDocument;
    }
  }

}
```

This approach is fairly idiomatic Java. In JavaScript or TypeScript the inner class could just be a class that is not exported from the module (a private class).

## Which to choose?

While this example may be trivial, it serves to demonstrate some of the strategies for dealing with mutable code. Which strategy you use is often not important, but it is important to limit the shared mutable state of an application to make it more deterministic and easier to reason about.
