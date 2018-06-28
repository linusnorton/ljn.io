---
title: A practical introduction to conditional types
date: 2018-06-28 06:00
lastmod: 2018-06-28 06:00
description: TypeScript 2.8 introduced conditional types, a powerful if esoteric feature.
layout: post
---

Conditional types were the headline feature in [TypeScript 2.8](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-2-8.html) but their use cases are not immediately obvious. The documentation describes a conditional type as:
```
... the ability to express non-uniform type mappings. A conditional type selects one of two possible types based
on a condition expressed as a type relationship test:

T extends U ? X : Y

The type above means when T is assignable to U the type is X, otherwise the type is Y.
```

For mere mortals, it is an if statement for types. It's certainly an interesting feature, so let's take a look at how we can use conditional types to write a HTTP request validator.

## A basic request HTTP request validator

Starting with a simple case we can define a request:

```typescript
interface Request {
  name: string,
  age: number,
  favouriteColour?: "red" | "blue" | "green",
  petsName?: string;
}
```

In this request the `name` and `age` must be set but the `favouriteColour` and `petsName` fields are optional. Our request handler will process a request from the HTTP framework (e.g. express/koa):

```typescript
function handler(request: Request) {
  const [firstName, lastName] = request.name.split(" ");
}
```

The issue with this is that we don't know whether our `handler` function is going to receive a complete request or not. If we were to be more accurate the `handler` would be defined as:

```typescript
function handler(request: Partial<Request>) {
  const [firstName, lastName] = request.name.split(" ");
}
```

This results in a compilation error because the type of `request.name` is now `string | undefined`, as `Partial<Request>` is:

```typescript
{
  name?: string,
  age?: number,
  favouriteColour?: "red" | "blue" | "green"
}
```

Our request validator can assert that the `Partial<Request>` we receive is actually a `Request` by checking that every mandatory property is not `undefined`:

```typescript
function isValid<T>(mandatory: string[], request: Partial<T>): request is T {
  return mandatory.every(field => request[field] !== undefined);
}
```

Now when our handler accesses the `name` property it knows it will be a `string` and not `undefined`:

```typescript
function handler(request: Partial<Request>) {
  if (!isValid(["name", "age"], request)) {
    throw new BadRequest(400, "Missing field");    
  }

  const [firstName, lastName] = request.name.split(" ");
}
```

Unfortunately, we have to explicitly pass the fields we want our validator to check as TypeScript doesn't maintain any type information at runtime (because it's just JavaScript!). However, we can add a little bit of type safety to our validator using conditional types to assert that the fields we receive are the non optional keys of `Request`.

```typescript
type NotUndefined<T> = Exclude<T, undefined>;
type MandatoryPropertiesNames<T> = { [K in keyof T]: T[K] extends NotUndefined<T[K]> ? K : never }[keyof T];

function isValid<T>(mandatory: MandatoryPropertiesNames<T>, request: Partial<T>): request is T {
  return mandatory.every(field => request[field] !== undefined);
}
```

The `NotUndefined<T>` type will exclude `undefined` from a list of types, so if we pass it `string | number | undefined` then the result will be `string | number`.

Using conditional types we can construct a list of mandatory properties of an object by asserting that the type of the key is the same as the as the type of the key without `undefined`:

```typescript
T[K] extends NotUndefined<T[K]>
```

If this statement holds, then the type of `K` is set to `K`, otherwise it is set to `never`:

```typescript
T[K] extends NotUndefined<T[K]> ? K : never
```

A type of `never` is omitted from the final object, so the result only contains the properties that cannot be set to `undefined`.

Now if we passed `["name", "petsName"]` as mandatory fields we'd get compilation error because `"petsName"` is not a mandatory field.

## Default properties

We may also want to populate the optional properties in our request with a default value:

```typescript
function withDefaults<T>(defaults: object, request: T): T {
  return Object.assign({}, defaults, request);
}
```

The type information here is not entirely accurate meaning our handler wouldn't know that the optional parameters have been set:

```typescript
function handler(request: Partial<Request>): Response {
  if (!isValid(["name", "age"], request)) {
    throw new BadRequest(400, "Missing field");    
  }

  const defaults = { favouriteColour: "red", petsName: "John" };
  const requestWithDefaults = withDefaults(defaults, request);
  const colour = requestWithDefaults.favouriteColour.toUpperCase();
}
```

The type of `requestWithDefaults.favouriteColour` is still `"red" | "blue" | "green" | undefined` so the call to `toUupperCase()` would generate an error.

We can tighten up the type information by being more specific about the default parameters and return type:

```typescript
type OptionalPropertiesNames<T> = { [K in keyof T]: T[K] extends NotUndefined<T[K]> ? never : K }[keyof T];
type OptionalProperties<T> = Pick<T, OptionalPropertiesNames<T>>;
type Complete<T extends object> = { [K in keyof T]-?: T[K]; };

function withDefaults<T>(defaults: Complete<OptionalProperties<T>>, request: T): Complete<T> {
  return Object.assign({}, defaults, request);
}
```

The `Complete<T>` type returns a version of `T` where all optional properties have been set to required. The return type of our function becomes `Complete<Request>`:

```typescript
{
  name: string,
  age: number,
  favouriteColour: "red" | "blue" | "green" ,
  petsName: string;
}
```

This means `favouriteColour` and `petsName` can now be used without having to check they've been set.

The `OptionalPropertiesNames<T>` type is the inverse of `MandatoryPropertiesNames<T>` - all the properties that can be set to `undefined`. We can use the in built `Pick` type to extract those properties from another type:

```typescript
type OptionalProperties<Request> = {
  favouriteColour?: "red" | "blue" | "green",
  petsName?: string;
}
```

Wrapping `OptionalProperties<Request>` with `Complete` means that the `defaults` value passed in to the `withDefaults` function must have **every** optional parameter set.

For instance passing `{ favouriteColour: "red" }` to `withDefaults` results in a compilation error because the default value for `petsName` was not given.

## The holes waiting to be filled

While this is all interesting, it's not bullet-proof.

First, our definition of `favouriteColour` states that the value must be `"red" | "blue" | "green"` but the value passed from the client might be anything, likewise with `age`, it should be a number but we're not checking that.

Second, `MandatoryPropertiesNames<T>[]` ensures that every value in the `mandatory` array is a mandatory key of `T`, but it does not ensure that the array contains every mandatory property. It's possible to only pass `["age"]` and the compiler won't complain.

Without runtime type information these issues are hard, if not impossible, to fix. It might be possible to in future to write a compiler [macro or template](https://github.com/Microsoft/TypeScript/issues/14419) that puts the type information into runtime JavaScript, but there's been no indication of that yet.
