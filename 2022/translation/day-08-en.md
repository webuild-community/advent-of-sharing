---
title: Runtime check with Literal Union Types in TypeScript
author: huytd
date: 12-01-2022
---

Literal Union Types are a very convenient way to limit a set of literal values (string literals), for example:

```typescript
type AllowedNames = "Cam" | "Xoài" | "Chuối"
```

However, all the fancy features in TypeScript exist at compile time, and after the JS code is built, it's done.

For example, here I want to use `AllowedNames` to write a function that only accepts one parameter with a value that matches one of the `Orange`, `Mango`, `Banana` values of the `AllowedNames` type above. Unfortunately, those values only exist at the type level, not at runtime.

Well, there is actually a way to solve this issue!


First, instead of defining `AllowedNames` directly from string literal values as before, we will create a const array containing those values, and then define the `AllowedNames` from that array, by using the `typeof` keyword like this: 


```typescript
const allowedNames = ["Cam", "Xoài", "Chuối"] as const;
type AllowedNames = typeof allowedNames[number];
//                = "Cam" | "Xoài" | "Chuối"
```

Then, from this method, we have the `allowedNames` at runtime as well as the `AllowedNames` type at the type level! All we need to do is write a function to check the value as required:


```typescript
const isNameAllowed = (input: unknown): input is AllowedNames => {
    return typeof input === 'string' && allowedNames.includes(input as AllowedNames);
};

const checkName = (input: string): AllowedNames => {
    if (isNameAllowed(input)) {
        return input;
    } else {
        throw new Error("Please use one of the allowed name!");
    }
}

console.log(checkName("Chuối")); // Output: "Chuối"
console.log(checkName("Cà Rốt")); // [ERR]: Please use one of the allowed name!
```

Awesome!

Wait, why does the code above run? What is `as const`? What is `typeof allowedNames[number]`???

This is where the brain-hurting part of Typescript begins.

First, what if we don't use the `as const` keyword?


```typescript
const allowedNames = ["Cam" | "Xoài" | "Chuối"];
type t = typeof allowedNames; // t == string[]
type t0 = typeof allowedNames[0]; // t0 == string
type t1 = typeof allowedNames[1]; // t1 == string
```

Very simple, `allowedNames` will be an array of `string`s, and each element in the `allowedNames` array will have be of `string` type. 

The `as const` (also known as [const assertion](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)) has the function of telling the Typescript compiler that the elements of the `allowedNames` array are `readonly`. What does this mean? It means that these elements can no longer change their values, so the type of each element in this array will definitely be literal types with its own value, instead of necessarily being a wider type like `string`. 


```typescript
const allowedNames = ["Cam" | "Xoài" | "Chuối"] as const;
type t = typeof allowedNames; // t == readonly ["Cam" | "Xoài" | "Chuối"]
type t0 = typeof allowedNames[0]; // t0 == "Cam"
type t1 = typeof allowedNames[1]; // t1 == "Xoài"
```

That's the `as const` part done. 

Next, like the two examples you can see above, we can get the type of an element by using the `typeof <Array>[index]` syntax. The use of `typeof <Array>[index]` is a type of [Indexed Access Type](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html), returning a union type containing **all possible types** of **every element** in the array. The reason why `number` is used is because we access elements the elements of the array using a `number` type index.  

Thus, `typeof allowedNames[number]` will return a union type containing the types of all elements in the `allowedNames` array, which are `"Cam" | "Xoài" | "Chuối"`.

Hopefully from this post, you also see the benefits of Typescript. Personally, I have already seen how it can really hurt my brain.
