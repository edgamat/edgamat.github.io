---
layout: post
title: 'Unsound TypeScript'
author: 'Matthew Edgar'
excerpt_separator: <!--more-->
---

Recently the team I work with has debated the unsound behaviors of TypeScript. I wanted to explore 3 areas where this has occurred for me in a recent project.

<!--more-->

### Sound versus Unsound 

Here's a good definition of soundness with respect to a programming language:

> Roughly speaking, a language is "sound" if the static type of every symbol is guaranteed to be compatible with its runtime value.

Source: https://effectivetypescript.com/2021/05/06/unsoundness/

TypeScript has decided (for better or worse) to allow unsound behavior to exist. The above article is a great discussion of these behaviors. Another good resource is the TypeScript Handbook:

https://www.typescriptlang.org/docs/handbook/type-compatibility.html#a-note-on-soundness

Theoretically, the unsound behavior is easy to demonstrate. In practice, the use of type assertions is one of the primary instances of unsound behavior. I'd like to demonstrate three of these areas and what can be done to avoid using them.

### DOM Objects

This is an example that was demonstrated to me recently:

```typescript
const button = document.querySelector("#run-button"); // button: Element | null
```

TypeScript thinks that `button` is of type `Element | null`, which is true (and sound). If the query finds a matching element, all it can infer is that is of type `Element`. However, TypeScript also allows you to do this at compile-time without raising a compiler warning:

```typescript
const button : HTMLButtonElement | null = document.querySelector("#run-button");
```

Although not immediately obvious, this is a type assertion. You've now told the compiler that you know more than it does. The `document.querySelector` function is a built-in JavaScript function, and as such, has no type safety beyond `Element | null`. But you've told the compiler that if found, the button will be of type `HTMLButtonElement` (At run-time this is actually an `HTMLAnchorElement` so this would potentially cause run-time errors).

The reason this is not a compile-time error has to do with how the `querySelector` has been described in the type definition file. When the selector is a known HTML element name, TypeScript can infer a more precise type for the element. 

Take this for example:

```typescript
const firstAnchor = document.querySelector("a"); // firstAnchor: HTMLAnchorElement | null
```

`firstAnchor` is of type `HTMLAnchorElement | null`. Because the selector `a` is one of the known HTML elements JavaScript has defined, TypeScript knows that if it finds anything, it will be of type `HTMLAnchorElement`. This allows it to narrow the type down from `Element` to a more precise type.

But due to the way the type definition file allows for this type of behavior, it is forced to allow type assertions to exist:

```typescript
const button1 : HTMLButtonElement | null = document.querySelector("#run-button");
const button2 = document.querySelector<HTMLButtonElement>("#run-button");
```

Neither of these statements will cause a compile-time issue because they are valid type assertions.

The `document.createElement` behaves in a similar way:

```typescript
const tagName1 = "a";
const anchor1 = document.createElement(tagName1); // anchor1: HTMLAnchorElement

const tagName2: string = "a";
const anchor2 = document.createElement(tagName2); // anchor2: HTMLElement
```

For `anchor1`, the tagName is a string literal containing one of the known HTML Element tag names. TypeScript narrows the inferred type. But for `anchor2`, the tagName can be any string, and as such, TypeScript doesn't infer anything beyond the base type of `HTMLElement`.

In all these situations, using a type guard will ensure you have the expected type:

```typescript
const button = document.querySelector("#run-button"); // button: Element | null
if (button instanceof HTMLAnchorElement) {
  console.log(button.href)
}
```

A type guard is a boolean expression that if holds true, allows TypeScript to infer a narrower type for a value. They are very useful in detecting unsound behavior at compile-time. 

When dealing with the DOM objects I see two recommendations:

1. Let the compiler infer the type of the object being returned. Anything you do to coerce the type is unsound.
2. Use type guards wherever possible to ensure the run-time values are what you think they should be.

### Database Results

When retrieving data from external sources, it is impossible to know what the results will be at run-time. Fetching data from a database is one such example. Here is a code sample using the `pg` library for Postgres (along with the `@types/pg` type definitions):

```typescript
try {
  const result = await client.query(
    `SELECT * FROM orders WHERE orderid = $1`,
    [orderId]
  );

  if (result.rowCount < 1) {
      return null;
  }

  return result.rows[0];
} finally {
  client.release();
}
```

TypeScript infers the type of `result` to be `QueryResult<any>` because it cannot know at compile time what the columns of the `orders` table are. The elements of the `rows` array are of type `any`. This is all sound behavior.

However, the library allows you to use an assertion on the type of data it returns via the `query` function:

```typescript
const result = await client.query<Order>(...)
```

But this is not sound because it is a type assertion. The recommendation here is to avoid using the type assertion. You can use a type guard or explicitly parse the results to know if the data is as you expect it to be.

### Axios HTTP Response Data

An almost identical situation occurs when using the results of calls made using the `axios` library (and the `@types/axios` type definitions):

```typescript
const response = await axios.get(url); // response: AxiosResponse<any>
```

This is sound because TypeScript is correctly inferring nothing about the response body. It sees it as an `any` type. But the `axios` type definitions allow you to introduce a type assertion:

```typescript
const response = await axios.get<Order>(url);
```

As with the previous examples, this is not sound because TypeScript cannot know if the data returned from the HTTP request is a JSON object matching the definition of `Order`. 

And the recommendation is the same. You should use a type guard or explicitly parse the results to know if the data is as you expect it to be.

### Summary

I have demonstrated a few places where you can inadvertently use unsound behavior with TypeScript. They all are using type assertions which can lead to run-time bugs/errors. There are several sound alternatives to ensure you know the type at run-time and avoid such problems. 
