---
title: TypeScript
---

## Type Annotations

Add explicit type information for variables, parameters, and return values.
JavaScript is dynamically typed, so type errors happen _at runtime_. TypeScript
catches these at **compile-time**, reducing runtime bugs.

When you write `let a: number`, TypeScript enforces that only values of type
`number` can be assigned. Under the hood, TypeScript doesn’t add types at
runtime — it removes them in the compiled JS — but the compiler ensures type
correctness before transpilation.

```ts
let age: number = 30;
function greet(name: string): string {
  return `Hello, ${name}`;
}
```

### Pitfalls

- Remember: annotations **don’t exist** at runtime — they don’t protect you from
  runtime `typeof` issues.
- Overusing explicit annotations when TypeScript can infer types makes code
  verbose.

## Interfaces

Define a **contract** (shape) for objects/classes. In large apps, especially
with API DTOs, they ensure that objects have the right structure and types.

Interfaces are erased at compile-time; they are purely a compile-time construct.
Multiple interfaces with the same name merge their definitions (interface
merging).

```ts
interface User {
  id: number;
  name: string;
  isAdmin?: boolean; // optional
}

const u: User = { id: 1, name: "Alice" };
```

```ts
interface Admin extends User {
  permissions: string[];
}
```

```ts
interface Greeter {
  (name: string): string;
}
```

### Pitfalls

- Interfaces can’t represent some advanced type constructs as well as type
  aliases (`type`), but they are better for object shapes and class contracts.

## Generics

Allow components/functions to work with **any type**, but still be type-safe.
Without generics, you’d use `any` — losing type safety. Generics retain
flexibility _and_ type awareness.

A generic type parameter `T` acts as a placeholder that is substituted when the
function/type is used.

```ts
function identity<T>(value: T): T {
  return value;
}

let str = identity<string>("Hello");
let num = identity(42); // type inferred
```

### Constraints

```ts
function getLength<T extends { length: number }>(value: T) {
  return value.length;
}
```

### Default type

```ts
function log<T = string>(value: T) {
  console.log(value);
}
```

## Enums

Represent a set of **named constants**. Improves readability, enforces valid
values, and reduces magic strings/numbers.

```ts
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

- **Numeric enums** auto-increment: `Up = 0, Down = 1, ...`
- **String enums** must have explicit values:

```ts
enum Role {
  Admin = "ADMIN",
  User = "USER",
}
```

### Pitfalls

- Enums create extra runtime objects, unlike literal unions (`type Role =
"ADMIN" | "USER"`) which are erased completely.
- Literal unions are often preferred for zero-runtime cost.

## Access Modifiers (public, private, protected)

Control visibility of class members. Encapsulation — hide internal details from
consumers.

```ts
class Car {
  private speed: number;
  protected brand: string;

  constructor(speed: number, brand: string) {
    this.speed = speed;
    this.brand = brand;
  }

  public accelerate(amount: number) {
    this.speed += amount;
  }
}
```

At runtime, these are not enforced (because JS doesn’t have true access
modifiers pre-ES2022 private fields), but the compiler will block invalid usage.

TypeScript also supports ES2022 `#private` fields, which are enforced at
runtime:

```ts
class Test {
  #secret = 123;
}
```

## Type Inference

Let TS deduce the type without explicit annotation. Less boilerplate, still
type-safe.

```ts
let x = 42; // inferred as number
const y = "abc"; // inferred as string literal type
```

In function parameters, TS never infers from usage — you must annotate if you
want a type:

```ts
function greet(name) {} // name = any, unless noImplicitAny = true
```

## Union & Intersection Types

Union is for _either/or_, Intersection is for _both_.

**Union (`|`)** — value can be one of several types.
**Intersection (`&`)** — combine multiple types into one.

```ts
let id: number | string;
id = 123;
id = "abc";

interface Name {
  name: string;
}
interface Age {
  age: number;
}
type Person = Name & Age; // must have both
```

## Non-null Assertion (`!`)

Tell TS _I know this is not null or undefined_. Useful when compiler can’t
guarantee non-nullness but you can. If you’re wrong, runtime crash.

```ts
const el = document.getElementById("app")!;
el.innerHTML = "Loaded!";
```

## Literal Types

Restrict variable to specific literal values. Good for finite config options or
state machine states.

```ts
let role: "admin" | "user";
role = "admin"; // ✅
role = "guest"; // ❌
```

## Conditional Types

A **type-level if-else** — lets you create a type based on a condition.

- If `T` is assignable to `U`, result is `X`, else `Y`.

```ts
T extends U ? X : Y
```

### Basic Example

```ts
type IsString<T> = T extends string ? "Yes" : "No";

type A = IsString<string>; // "Yes"
type B = IsString<number>; // "No"
```

### With Generics

```ts
function processValue<T>(value: T): T extends string ? string[] : T[] {
  return (typeof value === "string" ? value.split("") : [value]) as any;
}

const result1 = processValue("hi"); // string[]
const result2 = processValue(123); // number[]
```

### Key Use Cases

#### Type transformations

```ts
type ElementType<T> = T extends (infer U)[] ? U : T;
type E1 = ElementType<string[]>; // string
type E2 = ElementType<number>; // number
```

#### Excluding or extracting

```ts
type NonString<T> = T extends string ? never : T;
type Filtered = NonString<string | number | boolean>; // number | boolean
```

#### Function overload behavior

```ts
type ReturnTypeIfString<T> = T extends string ? number : boolean;
let example: ReturnTypeIfString<"abc">; // number
```

### Pitfalls

- Conditional types **distribute** over unions:

```ts
type Dist<T> = T extends string ? "yes" : "no";
type D = Dist<string | number>; // "yes" | "no"
```

- Use parentheses to avoid:

```ts
type NoDist<T> = [T] extends [string] ? "yes" : "no";
type ND = NoDist<string | number>; // "no"
```

## Mapped Types

Create new types by transforming **each property** in an existing type.

```ts
{ [K in Keys]: NewType }
```

### Basic Example

```ts
interface User {
  id: number;
  name: string;
}

type Stringify<T> = {
  [K in keyof T]: string;
};

type StringifiedUser = Stringify<User>;
// { id: string; name: string }
```

### Modifiers

- `readonly` → makes property immutable
- `?` → makes property optional
- `-readonly` / `-?` → removes readonly/optional

```ts
type Optional<T> = { [K in keyof T]?: T[K] };
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };
```

### With `keyof`

```ts
type Keys = keyof User; // "id" | "name"

type Nullable<T> = { [K in keyof T]: T[K] | null };
type NullableUser = Nullable<User>;
// { id: number | null; name: string | null }
```

### Key Remapping (`as`) _(TS 4.1+)_

```ts
type RenameKeys<T> = {
  [K in keyof T as `prefix_${string & K}`]: T[K];
};

type PrefixedUser = RenameKeys<User>;
// { prefix_id: number; prefix_name: string }
```

### Common Use Cases

#### Utility Types Implementation

- `Partial<T>`:

  ```ts
  type Partial<T> = { [K in keyof T]?: T[K] };
  ```

- `Omit<T, K>`:

  ```ts
  type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
  ```

#### Deep transformations

```ts
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
```

#### Dynamic API responses

```ts
type ApiResponse<T> = {
  [K in keyof T]: { value: T[K]; loaded: boolean };
};
```

## Combining Conditional + Mapped Types

### Make properties optional if they are functions

```ts
type OptionalFunctions<T> = {
  [K in keyof T]: T[K] extends Function ? T[K] | undefined : T[K];
};

interface Mixed {
  name: string;
  log(): void;
}

type Result = OptionalFunctions<Mixed>;
// name: string;
// log?: () => void
```

### Remove `null` only from string properties

```ts
type CleanStrings<T> = {
  [K in keyof T]: T[K] extends string | null ? string : T[K];
};

interface Data {
  id: number;
  title: string | null;
}

type Cleaned = CleanStrings<Data>;
// { id: number; title: string }
```

---

## Utility types

### `Partial<T>`

Make all properties in a type **optional**. Useful when updating objects — you
might only pass some fields.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

function updateUser(id: number, fields: Partial<User>) {
  // fields can have any subset of User properties
}

updateUser(1, { email: "new@mail.com" });
```

## `Required<T>`

Make all properties in a type **required** (opposite of `Partial`).

```ts
interface User {
  id?: number;
  name?: string;
}

type StrictUser = Required<User>;
```

### `Readonly<T>`

Make all properties **immutable**. Helps enforce immutability in functional
programming or Redux state.

```ts
interface User {
  id: number;
  name: string;
}

const u: Readonly<User> = { id: 1, name: "Alice" };
// u.id = 2; // ❌ Error
```

### `Pick<T, K>`

Select a subset of properties from a type. Useful for DTOs or views with limited
fields.

```ts
interface User {
  id: number;
  name: string;
  email: string;
}

type UserPreview = Pick<User, "id" | "name">;
```

### `Omit<T, K>`

Create a type by excluding certain properties. Opposite of `Pick`. Useful when
you want most but not all properties.

```ts
type UserWithoutEmail = Omit<User, "email">;
```

### `Record<K, T>`

Create an object type with keys `K` and values `T`. Useful for dictionary-like
structures.

```ts
type Roles = "admin" | "user";
type RolePermissions = Record<Roles, string[]>;

const perms: RolePermissions = {
  admin: ["read", "write"],
  user: ["read"],
};
```

### `Exclude<T, U>`

Remove from `T` all types assignable to `U`. Useful for filtering unions.

```ts
type Status = "active" | "inactive" | "deleted";
type ActiveStatus = Exclude<Status, "deleted">; // "active" | "inactive"
```

### `Extract<T, U>`

Keep only the types from `T` assignable to `U`.

```ts
type Status = "active" | "inactive" | "deleted";
type RemovedStatus = Extract<Status, "deleted">; // "deleted"
```

### `NonNullable<T>`

Remove `null` and `undefined` from a type.

```ts
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string
```
