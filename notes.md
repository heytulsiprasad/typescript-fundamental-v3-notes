# Typescript

URL: https://frontendmasters.com/courses/typescript-v2

### Typescript compiler

1.  Flags

To run → `tsc src/index.ts`

Run with target → `tsc src/index.ts --target ES2015`

Run with commonjs → `tsc src/index.ts --target ES2017 --module commonjs`

Run with watch → `tsc src/index.ts --target ES2017 --module commonjs --watch`

1. Config

```json
/* tsconfig.json */

{
  /* "files": ["src/index.ts"], */
  "include": ["src"], // supports blobs
  "compilerOptions": {
    "module": "commonjs",
    "target": "es2017",
    "outDir": "lib",
    "declaration": true, // adds a .d.ts consists of type declarations to outDir,
    "sourceMap": true // helps in debugging
  }
}
```

Most common ones:

![https://i.imgur.com/1q3RKx1.png](https://i.imgur.com/1q3RKx1.png)

**Note:** If you're just using TSC for checking and not producing the bundled output, coz you have webpack, babel and other stuff for that, you can set the target to `ESNext` and very well let your bundler take it from there.

---

## Typescript basics

```tsx
/**
 * (1) x is a string, b/c we’ve initialized it
 * @internal
 */
let x = "hello world";
```

- `internal` flags leave out the type definition from .d.ts file.
- `any` in type-systems in general is called a, top-type
- If variable declarations and initializations are separated, we can use the **type annotation** syntax (basically how flow does it)

```jsx
let z: number;
z = 41;
z = "abc"; // (6) oh no! This isn't good
```

- Use tuples which is an array of specific length, but can be initialized with specific types.

```jsx
let bb: [number, string, string, number] = [
  123,
  "Fake Street",
  "Nowhere, USA",
  10110,
];

bb = [1, "2", "3", 45];
```

- Use object types same way we do in flow, importantly we can use interfaces which is a TS thing that we can reuse everytime we define the same object.
- We can use **Intersection types** if we have some key-value pairs in an interface are optional under some conditions, then we could only access the k-v pairs which exist in all conditions.
- We can use **Union types** if we have an interface which has k-v pairs of both types.
- In object types, `?` means optional.

**Type systems**

- Nominal Type Systems → `x` is an instance of a class/type named as such
- Structural Type Systems (one that TS uses) → it only cares about the structure (shape) of the object

![https://i.imgur.com/NtdF7qk.png](https://i.imgur.com/NtdF7qk.png)

- Return types in functions can be inferred or can also be explicit.
- Function signatures are basically interesting. It gives meaning to what arguments are passed inside a function (instead of mere type-checking).

  ✏️ This is mostly used with functions with more than 1 arguments.

```tsx
// overload signatures
function contactPeople(method: "email", ...people: HasEmail[]): void;
function contactPeople(method: "phone", ...people: HasPhoneNumber[]): void;

// function implementation
function contactPeople(
  method: "email" | "phone",
  ...people: (HasEmail | HasPhoneNumber)[]
): void {
  if (method === "email") {
    (people as HasEmail[]).forEach(sendEmail);
  } else {
    (people as HasPhoneNumber[]).forEach(sendTextMessage);
  }
}

// ✅ email works
contactPeople("email", { name: "foo", email: "" });

// ✅ phone works
contactPeople("phone", { name: "foo", phone: 12345678 });

// 🚨 mixing does not work
contactPeople("email", { name: "foo", phone: 12345678 });
```

---

## Interfaces and Type aliases

- Extends can be used to extend pre-existing interfaces and classes [1]
- TS code checks code sequentially so we cant do cyclic assignments of types
- Interface are pretty much limited to things that have prototypes (but type annotations can do almost everything)
- We can use interfaces and type aliases to define a function signature (helpful in callbacks) [2]
- If a function has its signature, we could getaway without mentioning types in a function for its arguments, its called **contextual inference**
- Constructors can also have interfaces [3]

```tsx
// [1] Extend an interface
interface HasInternationalPhoneNumber extends HasPhoneNumber {
  countryCode: string;
}
```

```tsx
// [2] Function signature using interface
interface ContactMessenger1 {
  (contact: HasEmail | HasPhoneNumber, message: string): void;
}

// Function signature using types
type ContactMessenger2 = (
  contact: HasEmail | HasPhoneNumber,
  message: string
) => void;

// Don't need type annotations for contact or message anymore
// This is called `contextual inference`
const emailer: ContactMessenger2 = (contact, message) => {
  /* ... */
};
```

```tsx
// [3] Constructor interfaces
interface ContactConstructor {
  new (...args: any[]): HasEmail | HasPhoneNumber;
}
```

- Interfaces are open, meaning declarations of same name are merged together
- Interfaces are like functions they're hoisted and can allow cyclic reference (but also types can do that) but as per instructor, types are lazily loaded ⚠️

---

## Classes

- `implements` is called **Heritage clause**

Note: `extends` is only used when we're inheriting from another entity in javascript, but TS introduces `implements` which means aligning with a particular interface

```tsx
export class Contact implements HasEmail {
  email: string;
  name: string;
  constructor(name: string, email: string) {
    this.email = email;
    this.name = name;
  }
}
```

- Access modifiers are bread and butter for making classes.

1. public - Everyone
2. protected - me and subclass
3. private - only me

```tsx
class Contact implements HasEmail {
  constructor(public name: string, public email: string = "No email") {}
}
```

- Use `readonly` flag infront of a variable to mention your linter to scream if you touch it
- Class fields can have initializers (defaults)
- **Definite assignment operator** means you tell TS to allow you to initialize a variable without assigning it a value.

[0]: Property 'password' has no initializer and is not definitely assigned in the constructor.ts(2564)

[1]: ✅

[2]: Ambient declaration (used in type definition files)

[3]: Definite assignment assertion (property of type `string | undefined` ie undefined at first but is treated as string)

Source ⇒ [Differences explanation](https://stackoverflow.com/a/67354246/11674552)

```tsx
class OtherContact implements HasEmail, HasPhoneNumber {
  protected age: number = 0;
  private password: string; // Gives error [0]
  // private password: string | undefined; [1]
  // private declare password: string; [2]
  // private password!: string; [3]
  constructor(public name: string, public email: string, public phone: number) {
    // () password must either be initialized like this, or have a default value
  }
}
```

- Class can **implement** only interfaces, but it can **extend** other classes
- Abstract classes can be used to define a structure (or partial implementation) of its non-abstract subclasses.

```tsx
abstract class AbstractContact implements HasEmail, HasPhoneNumber {
  public abstract phone: number; // must be implemented by non-abstract subclasses

  constructor(
    public name: string,
    public email: string // must be public to satisfy HasEmail
  ) {}

  abstract sendEmail(): void; // must be implemented by non-abstract subclasses
}
```

Now if it's subclass won't contain `sendEmail` or inherit both name, email TS will throw an error.

```tsx
class ConcreteContact extends AbstractContact {
  constructor(
    name: string,
    email: string,
    public phone: number // must happen before non property-parameter arguments
  ) {
    super(name, email);
  }
  sendEmail() {
    // mandatory!
    console.log("sending an email");
  }
}
```

- Also checks for type returned by `sendEmail`. But here, when void is returned it's doesn't check anymore for types.

---

### Converting to TS

1️⃣ Rename files from js to ts, will make everything implicit any.

2️⃣ Use `noImplicit` as `true` in tsconfig

3️⃣ Import library types from DefinitelyTyped

4️⃣ `strictFunctionTypes`, `strictNullChecks`, `strict`

Note: Don't change code, change types

---

## Generics

Generics parameterize types as functions parameterize values

- Basically we can pass a type into an interface dynamically and use it to calculate some other stuff inside it

```tsx
interface DynamicInterface<T> {
  value: T;
}

// Usage
let flatNumber: DynamicInterface<number> = { value: 34 };
```

- They can have default types as well

```tsx
interface FilterFunction<T = any> {
  (val: T): boolean; // boolean is the output of numberFilter
}

// Usage
const numberFilter: FilterFunction<number> = (val) => typeof val === "number";
```

> "Generic over something" means it abstracts to the type that it resolves to. Eg: Promise that resolves to number/HTTP response

```tsx
function resolveOrTimeout<T>(promise: Promise<T>, timeout: number): Promise<T> {
  return new Promise<T>((resolve, reject) => {
    // start the timeout, reject when it triggers
    const task = setTimeout(() => reject("time up!"), timeout);

    promise.then((val) => {
      // cancel the timeout
      clearTimeout(task);

      // resolve with the value
      resolve(val);
    });
  });
}
```

If we call this passing fetch, `T` resolve to `<Response>` as fetch has its own response type defined.

Fetch's own types: `function fetch(input: RequestInfo, init?: RequestInit | undefined): Promise<Response>`

Our function after T is passed: `function resolveOrTimeout<Response>(promise: Promise<Response>, timeout: number): Promise<Response>`

- Type parameters can also have scopes like function arguments
- Type parameters can have contraints

> By constraints I mean, you can extend a generic type T (to atleast have some parameters than others)

```tsx
function arrayToDict<T extends { id: string }>(array: T[]): { [k: string]: T } {
  const out: { [k: string]: T } = {};

  array.forEach((val) => {
    out[val.id] = val;
  });

  return out;
}

const myDict = arrayToDict([
  { id: "a", value: "first", lisa: "Huang" },
  { id: "b", value: "second" },
]);
```

- The way to see generic and type passing is

> **You ask for what you need and return everything you can**

- Generics are necessary when we want to describe a relationship between two or more types (ie the function argument and return type)
- But if it's (a type parameter) used only once it can probably be eliminated.

```tsx
interface Shape {
  draw();
}

interface Circle extends Shape {
  radius: number;
}

function drawShapes1<S extends Shape>(shapes: S[]) {
  shapes.forEach((s) => s.draw());
}

// This is same as the above function's parameters
function drawShapes2(shapes: Shape[]) {
  shapes.forEach((s) => s.draw());
}
```

## Top and Bottom Types

- TS has two top types: `any` and `unknown`
- They both can contain all values whatsoever but vary in their usecases
- `unknown` needs to be narrowed down to some type before actually using it
- `any` can be used where we don't care about the type and need **maximum flexibility** in case of a Promise where we don't care about returned value
- `unknown` is good for use in private values that aren't supposed to be a part of Public API

```tsx
let myUnknown: unknown = "hello, unknown";

if (typeof myUnknown === "string") {
  myUnknown.split(", "); // ✅ OK
}

if (myUnknown instanceof Promise) {
  myUnknown.then((x) => console.log(x));
}
```

- We can create custom typeguards ie functions that return boolean before using `unknown`

```tsx
function isDefined<T>(arg: T | undefined): arg is T {
  return typeof arg !== "undefined";
}

const list = ["Apples", undefined, "Mangoes"]; \\ (string | undefined)[]
const filteredList = list.filter(isDefined); \\ string[]
```

- However `unknown` can get dangerous as you can getaway with assignining unknown values around your code to each other.

```tsx
let aNumber: unknown = 41;
let aArray: unknown = ["a", "string", "array"];
aNumber = aArray;
```

- `never` is a type that means the value can never be reached in the code

```tsx
class UnreachableError extends Error {
  constructor(val: never, message: string) {
    super(`TypeScript thought we could never end up here\n${message}`);
  }
}

let y = 4 as string | number;

if (typeof y === "string") {
  y.split(", ");
} else if (typeof y === "number") {
  y.toFixed(2);
} else {
  // if we access y here, it's type would be `never`
  throw new UnreachableError(y, "y should be a string or number");
}
```

> The above example is called an exhaustive switch and exhaustive conditional.

Mike's two favorite functions to have while coding in TS:

- `isDefined` function
- `UnreachableError` class

## Advanced Types

Yet to study
