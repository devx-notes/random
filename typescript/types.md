# Types in TS

## 1. Basic Types 

- 
- 
- 

### Union Types

A union type is a type formed from two or more other types, representing values that may be any one of those types. 

```ts
function printId(id: number | string) {
  console.log("Your ID is: " + id);
}
```

Even though `typeof null` returns `object` in JavaScript, TypeScript's type system treats `null` as its own distinct type. In TypeScript, `T | null`  means that the variable can hold a value of type T or it can hold the null value. 

-  When [strictNullChecks](https://www.typescriptlang.org/docs/handbook/2/basic-types.html#strictnullchecks) is enabled, `null` and `undefined` have their own distinct types

```ts
interface MyObject {
  name: string;
}

function processObject(obj: MyObject | null) {
  if (obj !== null) {
    console.log(obj.name);
  } else {
    console.log("Object is null");
  }
}
```

### Intersection Types

An intersection type is defined using the & operator. It is mainly used to combine existing object types.

```ts
interface Colorful {
  color: string;
}
interface Circle {
  radius: number;
}
 
type ColorfulCircle = Colorful & Circle;
```

## 2. Object Types

Object type can be anonymous or can be named by using either an interface or a type alias

- [Differences between Type-aliases and Interfaces](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#differences-between-type-aliases-and-interfaces)
- [Cheatsheet](https://www.typescriptlang.org/cheatsheets/)

```ts
function greet(person: { name: string; age: number }) {
  return "Hello " + person.name;
}
```

An interface can be re-opened to add new 

```ts
interface Person {
  name: string;
  age: number;
}

interface Person{
    gender?: string
}
```

Almost all features of an interface are available in type, the key distinction is that a type cannot be re-opened to add new properties (only extending via intersections). 

```ts
type Person = {
  name: string;
  age: number;
};
 
function greet(person: Person) {
  return "Hello " + person.name;
}
```


