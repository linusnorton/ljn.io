---
title: Nominal typing in TypeScript
date: 2019-09-06 08:00:00
lastmod: 2019-09-06 08:00:00
description: >-
  Using nominal typing to implement identity types in TypeScript.
layout: post
---

Microsoft has recently been discussing a [proposal for nominal typing in TypeScript](https://github.com/microsoft/TypeScript/pull/33038), which will add the ability to make a type "unique".

At present (TypeScript 3.6) the compiler will treat two different types as the same if they logically equate to the same thing. For example:

```javascript
type OrderID = number;
type CustomerID = number;

function getCustomer(customerId: CustomerID) {
  return get("/customers/" + customerId);
}

const orderId: OrderID = 1;

getCustomer(orderId); // this is okay
```

This approach is called "[Duck Typing](https://en.wikipedia.org/wiki/Duck_typing)". If it looks like a duck and it quacks, then it must be okay.

There's nothing wrong with this approach, in most cases it's very useful, but as with the example above it can also be useful to explicitly define that types are not equal even if they look the same.

A common example of this is the [identity type pattern](https://medium.com/@gara.mohamed/domain-driven-design-the-identifier-type-pattern-d86fd3c128b3), where the identity of an entity is represented by a [Value Object](https://en.wikipedia.org/wiki/Value_object). There can be [many reasons to do this](https://buildplease.com/pages/vo-ids/) but I personally find it useful as it makes the code more descriptive and easier to refactor.

# Implementations

This is a common pattern in the Scala community, where identity classes can be implemented as a [Value Class](https://dzone.com/articles/what-value-classes-is). In Java it's just implemented as an immutable class. Both of these methods work well for type checking but add a slight overhead as additional code is needed to extract the value from the class that wraps it. The TypeScript equivalent would be a class with a "brand":

```javascript
class OrderID {
  private _brandOrderId: void;
  constructor(public readonly id: number) {}
}
class CustomerID {
  private _brandCustomerId: void;
  constructor(public readonly id: number) {}
}

function getCustomer(customerId: CustomerID) {
  return get("/customers/" + customerId.id); // value needs to be extracted
}

const orderId = new OrderID(1);

getCustomer(orderId); // compilation error
```

In TypeScript it's most common to just use type aliases and live without the compiler safety:

```javascript
type OrderID = number;
type CustomerID = number;
```

Through a process called nominal typing people are trying to get the best of both worlds by making the types unique, without having to wrap them in an extraneous class. There are [several approaches](https://michalzalecki.com/nominal-typing-in-typescript/) to this but they commonly revolve around mixing in a unique type:

```javascript
interface Identity<T extends string> {
  private as: T;
}

type OrderID = string & Identity<"OrderID">;
type CustomerID = string & Identity<"CustomerID">;

function getCustomer(customerId: CustomerID) {
  return get("/customers/" + customerId);
}

const orderId = 1 as OrderID; // needs to be cast to the OrderID type

getCustomer(orderId); // compilation error
```

Using this approach achieves the simplicity of aliasing as string but the type safety of creating a new class. The only downsides are the extra noise when defining the class and creating new variables with those types.

# The unique keyword

The [proposal](https://github.com/microsoft/TypeScript/pull/33038) to add unique as a keyword will solve both of these problems:

```javascript
type OrderID = unique number;
type CustomerID = unique number;

function getCustomer(customerId: CustomerID) {
  return get("/customers/" + customerId); // don't need to extract the value
}

const orderId: OrderID = 1; // does not need to be type cast

getCustomer(orderId); // compilation error
```

The proposal is still being discussed and it is unlikely to land for many months, but it seems like a clean solution that will make identity types easier to work with. Let's hope it lands.
