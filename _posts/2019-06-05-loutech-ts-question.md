---
layout: post
title: "TypeScript Mapped & Lookup Type Usage"
---

A fun TypeScript question was recently posed in the [LouisvilleTech Slack](https://louisvilletech.org/).

# Question, round 1

> In TypeScript, how can I declare a function so that it requires an argument to be a key of a certain type, and also that the key has a value of a certain type?

```TypeScript
interface Test {
  someKey: string;
};

fn<Test>({ someKey: 'someValue' }, 'nonexistentKey'); // shows error, which is correct because that key doesn't exist for that interface.
fn<Test>({ someKey: 'someValue' }, 'someKey'); // works, that is a valid key for that interface
fn<Test>({ someKey: 123 }, 'someKey'); // also works, but I don't want it to because the value should be a string
```

# Solution, round 1

```TypeScript
function sample<T extends { [key: string]: string }, K extends keyof T>(obj: T, key: K) {
    // ...
}

sample({ abc: 'def ' }, 'abc'); // works
sample({ abc: 'def ' }, 'efg'); // error, missing key
sample({ abc: 123 }, 'abc'); // error, wrong type
```
[Run in TypeScript Playground](https://www.typescriptlang.org/play/#src=function%20sample%3CT%20extends%20%7B%20%5Bkey%3A%20string%5D%3A%20string%20%7D%2C%20K%20extends%20keyof%20T%3E(obj%3A%20T%2C%20key%3A%20K)%20%7B%0D%0A%20%20%20%20%2F%2F%20...%0D%0A%7D%0D%0A%0D%0Asample(%7B%20abc%3A%20'def%20'%20%7D%2C%20'abc')%3B%20%2F%2F%20works%0D%0Asample(%7B%20abc%3A%20'def%20'%20%7D%2C%20'efg')%3B%20%2F%2F%20error%2C%20missing%20key%0D%0Asample(%7B%20abc%3A%20123%20%7D%2C%20'abc')%3B%20%2F%2F%20error%2C%20wrong%20type)

# Question, round 2

> Gotcha, that works by limiting the interface, but I wonder if there’s a way to allow all interfaces and only limit the keys you can enter for the second arg.

# Solution, round 2

This is definitely workable, and buried in the [TypeScript Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html) docs. Those aren't the easiest to digest, so here's a complete solution:

```TypeScript
type NamesOfStringProps<Type> = {
    [Key in keyof Type]: string extends Type[Key] ? Key : never
}[keyof Type];


type PickStrings<Type> = {
    [Key in NamesOfStringProps<Type>]: Type[Key]
};

function only_string_props<T, U extends PickStrings<T>, K extends keyof U>(obj: T, key: K) {
    // ...
}

let x = {
    alpha: 'a',
    beta: 'b',
    123: 456
};
only_string_props(x, 'alpha'); // works
only_string_props(x, 123); // error, not a key in the narrowed type
only_string_props(x, 'missing'); // error, missing key
```
[Run in TypeScript Playground](https://www.typescriptlang.org/play/#src=type%20NamesOfStringProps%3CType%3E%20%3D%20%7B%0D%0A%20%20%20%20%5BKey%20in%20keyof%20Type%5D%3A%20string%20extends%20Type%5BKey%5D%20%3F%20Key%20%3A%20never%0D%0A%7D%5Bkeyof%20Type%5D%3B%0D%0A%0D%0A%0D%0Atype%20PickStrings%3CType%3E%20%3D%20%7B%0D%0A%20%20%20%20%5BKey%20in%20NamesOfStringProps%3CType%3E%5D%3A%20Type%5BKey%5D%0D%0A%7D%3B%0D%0A%0D%0Afunction%20only_string_props%3CT%2C%20U%20extends%20PickStrings%3CT%3E%2C%20K%20extends%20keyof%20U%3E(obj%3A%20T%2C%20key%3A%20K)%20%7B%0D%0A%20%20%20%20%2F%2F%20...%0D%0A%7D%0D%0A%0D%0Alet%20x%20%3D%20%7B%0D%0A%20%20%20%20alpha%3A%20'a'%2C%0D%0A%20%20%20%20beta%3A%20'b'%2C%0D%0A%20%20%20%20123%3A%20456%0D%0A%7D%3B%0D%0Aonly_string_props(x%2C%20'alpha')%3B%20%2F%2F%20works%0D%0Aonly_string_props(x%2C%20123)%3B%20%2F%2F%20error%2C%20not%20a%20key%20in%20the%20narrowed%20type%0D%0Aonly_string_props(x%2C%20'missing')%3B%20%2F%2F%20error%2C%20missing%20key)

It works! But isn't exactly readable. Let's break this down in to discrete steps that are easier to reason about.

```TypeScript
let x = {
    alpha: 'a',
    beta: 'b',
    123: 456
};

// get type of our object
type T = typeof x;

// Make a mapped type over `T`
// 1. string props pass through with their values
// 2. other props get value set to `never`
type StringValuesOrNever = {
    [K in keyof T]: T[K] extends string ? K : never
};

// Lookup type over the mapped type
// `keyof` skips props with `never` values :wand:
type KeysOfStringProps = StringValuesOrNever[keyof T];

// Use built-in `Pick` type with keys of string props
// Leaves us with type with only string props of `x`
type TypeWithOnlyStringProps = Pick<T, KeysOfStringProps>;
```
[Run in TypeScript Playground](https://www.typescriptlang.org/play/#src=let%20x%20%3D%20%7B%0D%0A%20%20%20%20alpha%3A%20'a'%2C%0D%0A%20%20%20%20beta%3A%20'b'%2C%0D%0A%20%20%20%20123%3A%20456%0D%0A%7D%3B%0D%0A%0D%0A%2F%2F%20get%20type%20of%20our%20object%0D%0Atype%20T%20%3D%20typeof%20x%3B%0D%0A%0D%0A%2F%2F%20Make%20a%20mapped%20type%20over%20%60T%60%0D%0A%2F%2F%201.%20string%20props%20pass%20through%20with%20their%20values%0D%0A%2F%2F%202.%20other%20props%20get%20value%20set%20to%20%60never%60%0D%0Atype%20StringValuesOrNever%20%3D%20%7B%0D%0A%20%20%20%20%5BK%20in%20keyof%20T%5D%3A%20T%5BK%5D%20extends%20string%20%3F%20K%20%3A%20never%0D%0A%7D%3B%0D%0A%0D%0A%2F%2F%20Lookup%20type%20over%20the%20mapped%20type%0D%0A%2F%2F%20%60keyof%60%20skips%20props%20with%20%60never%60%20values%20%3Awand%3A%0D%0Atype%20KeysOfStringProps%20%3D%20StringValuesOrNever%5Bkeyof%20T%5D%3B%0D%0A%0D%0A%2F%2F%20Use%20built-in%20%60Pick%60%20type%20with%20keys%20of%20string%20props%0D%0A%2F%2F%20Leaves%20us%20with%20type%20with%20only%20string%20props%20of%20%60x%60%0D%0Atype%20TypeWithOnlyStringProps%20%3D%20Pick%3CT%2C%20KeysOfStringProps%3E%3B)

With all that done, and hopefully clear, there is actually a much terser version using the built-in utility types, `Pick` and `Extract`:
```TypeScript
type PickOfType<T, P> = Pick<T, Extract<keyof T, P>>;

function only_string_props<T, K extends keyof PickOfType<T, string>>(obj: T, key: K) {
    // ...
}

let x = {
    alpha: 'a',
    beta: 'b',
    123: 456
};

only_string_props(x, 'alpha') // works
only_string_props(x, '123') // error
```
[Run in TypeScript Playground](https://www.typescriptlang.org/play/#src=type%20PickOfType%3CT%2C%20P%3E%20%3D%20Pick%3CT%2C%20Extract%3Ckeyof%20T%2C%20P%3E%3E%3B%0D%0A%0D%0Afunction%20only_string_props%3CT%2C%20K%20extends%20keyof%20PickOfType%3CT%2C%20string%3E%3E(obj%3A%20T%2C%20key%3A%20K)%20%7B%0D%0A%20%20%20%20%2F%2F%20...%0D%0A%7D%0D%0A%0D%0Alet%20x%20%3D%20%7B%0D%0A%20%20%20%20alpha%3A%20'a'%2C%0D%0A%20%20%20%20beta%3A%20'b'%2C%0D%0A%20%20%20%20123%3A%20456%0D%0A%7D%3B%0D%0A%0D%0Aonly_string_props(x%2C%20'alpha')%20%2F%2F%20works%0D%0Aonly_string_props(x%2C%20'123')%20%2F%2F%20error)

Pay special attention to those repeated `keyof` expressions, that's the secret sauce here.
