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

# Question, round 2

> Gotcha, that works by limiting the interface, but I wonder if there’s a way to allow all interfaces and only limit the keys you can enter for the second arg.

# Solution, round 2

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

Let's break this down in to discrete steps that are easier to digest.

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
