# Level up on TypeScript! What I learned from type-challenges

# Summary of this article

This article introduces type-challenges, which helped me(a TypeScript beginner) to better understand TypeScript.  <br/>
type-challenges is a collection of TypeScript challenges divided into levels. You can learn while having fun!

In the first half of this article, I will introduce what kind of TypeScript challenges you will face with type-challenges's category "easy".<br/>
And also I will introduce what you can learn from them.

In the second half, I will briefly explain the techniques I often used to solve the challenges.

Here is the link to type-challenges.<br/>
https://github.com/type-challenges/type-challenges

# type-challenge category "easy"

## What kind of challenges

There are 13 challenges in total in the category "easy". (at the time of writing)<br/>
The challenge is to implement the following utility types.

- `Pick<T, K>` utility type (4 - Pick)
- `Readonly<T>` utility type (7 - ReadOnly)
- `TupleToObject<T>` utility type, which converts an array to an object type,  (11 - Tuple to Object)
- `First<T>` utility type, which returns the first element of an array (14 - First)
- `length<T>` utility type, which returns the length of an array (18 - Length of Tuple)
- `Exclude<T, U>` utility type (43 - Exclude)
- `Awaited<T>` utility type (189 - Awaited)
- `If<C, T, F>` utility type, which returns `T` if `C` is truthy, and returns`F` if falsy) (268 - If)
- Type version of `Array.concat()` (533 - Concat)
- Type version of `Array.includes()` (898 - Includes)
- Type version of `Array.push()` (3057 - Push)
- Type version of `Array.unshift()` (3060 - Unshift)
- `Parameters<T>` utility type (3312 - Parameters)

## Techniques you can learn

First of all, Generics is used in all challenges, so it is great for those who want to practice Generics.<br/>
Below are some of the techniques often used to solve the challenges.<br/>
I will only introduce the terminology here and explain them in the following section.

- Generic Constraints
- `readonly` modifier
- Mapped Types
- Indexed Access Types
- Conditional types
  - Filtering by `never`
  - Type inference with `infer`
- Variadic Tuple Types

If you have any terminology that makes you wonder "what the heck is that 🤔", try type-challenges!

# Frequently used techniques

Here is a brief description of the commonly used techniques in the challenges of  type-challenges category "easy".<br/>
This is only a brief explanation, so please check the reference links for details.

## Generic Constraints

Generic Constraints is a way to restrict Generics to a specific type using the `extends` keyword.

This is useful, for example, in the following example.

*Example: `arg` with type Generics `T` calls the `length` method, but since Generics can be arbitrary (it may not have `length` method), a compile error occurs. *

```ts
function checkLength<T>(arg: T): number {
  // Error: Property 'length' does not exist on type 'T'.
  return arg.length;
}
```

In such cases, the `extends` keyword can be used to constrain the type of Generics type arguments to a specific type. (Generic Constraints)<br/>
This allows you to safely access the properties of an object.

```ts
function checkLength<T extends string>(arg: T): number {
  return arg.length;
}
```

Reference: 
https://www.typescriptlang.org/docs/handbook/2/generics.html#generic-constraints

### Combination with `keyof` type operators

Generic Constraints is often used in combination with `keyof` type operators. (See the reference link below for more information on `keyof`.)<br/>
In the following example, the second argument `key` of the `getProperty` function is set to `K extends keyof T` to restrict the type to accept only those objects of type `T` (union type of the `x` object's property key).

```ts
function getProperty<T, K extends keyof T>(obj: T, key: K) {
  return obj[key];
}
// K => "a" | "b" | "c" | "d"
 
let x = { a: 1, b: 2, c: 3, d: 4 }
 
getProperty(x, "a");
getProperty(x, "m");
// Error: Argument of type '"m"' is not assignable to parameter of type '"a" | "b" | "c" | "d"'.
```
Reference: 
https://www.typescriptlang.org/docs/handbook/2/keyof-types.html

## `readonly` modifier

The `readonly` modifier can be used to make arrays or object properties immutable.

*Example: assignment to `obj` foo causes an error.*
```ts
let obj: {
  readonly foo: number;
};
obj = { foo: 1 }
obj.foo = 2;
// Error: Cannot assign to 'foo' because it is a read-only property.
```

*Example: Assigning to `a` or changing an element causes an error.*
```ts
let a = readonly ['a', 'b', 'c'];
a.push('d'); // error
a[0] = 'x'; // error
```

Reference: 
https://www.typescriptlang.org/docs/handbook/typescript-in-5-minutes-func.html#readonly-and-const

## Mapped Types

Mapped Types is a technique for creating types by iterating over keys by using the `in` keyword on a union type.<br/>
The following example creates a `HelloTranslation` object type using the `in` keyword for the `AvialbleLanguage` union type.

```ts
type AvailableLanguage = 'en' | 'jp';

type HelloTranslation = {
    [key in AvailableLanguage]: string;
}
/**
 * type HelloTranslation = {
 * en: string; }
 * jp: string; }
 * }
 */

const hello: HelloTranslation = {
    'en': 'hello', 
    'jp': 'hello',
    'fr': 'bonjour',
    /**
     * Error: Type '{ en: string; jp: string; fr: string; }' is not assignable to type 'HelloTranslation'.
     * Object literal may only specify known properties, and ''fr'' does not exist in type 'HelloTranslation'.
     */       
}
```

To map to union types, another technique often used is to use the `keyof` type operator to turn properties of an object type into union types, and then iterate over them with `in`.

Reference: 
https://www.typescriptlang.org/docs/handbook/2/mapped-types.html#handbook-content

## Indexed Access Types

Indexed Access Types are similar to the way JavaScript accesses object properties.<br/>
The following example accesses property types of a `Cat` object type.

```ts
type Cat = { age: number; name: string; alive: boolean }
type Age = Cat["age"];
// type Age = number;
```

The index type to be accessed is itself a type, so you can also use union types to access multiple properties.

````ts
type Cat = { age: number; name: string; alive: boolean }

type Type1 = Cat["age" | "name"];  
// type Type Type1 = string | number
type Type2 = Cat[keyof Cat];
// type Type Type2 = string | number | boolean
````

Another useful technique for Indexed Access Type is to access tuple types with `[number]`.
This allows elements of a tuple type to be retrieved as union types.

```ts
type Animals = ['cat', 'dog', 'tiger', 'lion', 'elephant'];

type AnimalsUnion = Animals[number];
// type AnimalsUnion = 'cat' | 'dog' | 'tiger' | 'lion' | 'elephant'
````

This is a useful technique that is powerful when combined with the Mapped types above because it generates union types.

Reference: 
https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html

## Conditional types

Conditional types are similar to JavaScript's Ternary operator using `extend` and `? ` to create conditional types.

```ts
type True = true extends boolean ? true : false;
```

Reference: 
https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#handbook-content

### Filtering with `never`

You can use `never`, meaning a value that cannot occur, as a return for Conditional types to narrow down the types.<br/>
In the following example, elements of union types not in `AvailableLanguage` are excluded by `never`.

```ts
type AvailableLanguage = 'en' | 'jp';
type FilteredLanguage<T> = T extends AvailableLanguage ? T : never;
type Foo = FilteredLanguage<'en' | 'fr' | 'jp' | 'cn' | 'kr'>; 
// type Foo = "en" | "jp"
```

Reference: 
https://www.typescriptlang.org/docs/handbook/2/functions.html#never

### type inference with `infer

`infer` is used with Conditional types to infer the type that matches a condition.
The following example shows how to infer the return type of a function and get its return value with `infer` if `T` is a function type.

```ts
type GetReturnType<T> = T extends (. .args: unknown[]) => infer Return
  ? Return
  : never;

type Num = GetReturnType<() => number>;
// type Num = number 

type Str = GetReturnType<(x: string) => string>;     
// type Str = string
 
type Bools = GetReturnType<(a: boolean, b: boolean) => boolean[]>;   
// type Bools = boolean[]
```

Reference: 
https://www.typescriptlang.org/docs/handbook/2/conditional-types.html#inferring-within-conditional-types

## Variadic Tuple Types

Variadic Tuple Types are like type versions of JavaScript's spread syntax.

```ts

type Numbers = [number, number];
type StrStrNumNumBool = [. .Strings, . .Numbers, boolean];
// type StrStrNumNumBool = [string, string, number, number, boolean]
```

Reference: 
https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-0.html#variadic-tuple-types

# Summary

This article summarizes what you can learn with type-challenges category "easy".<br/>
If you are looking for a way to learn TypeScript, I recommend type challenges, a hands-on, fun way to learn!
Each challenge has a well-explained solution page, so give it a shot!

The article is also available in Japanese: https://zenn.dev/takuyakikuchi/articles/19d8769895a863