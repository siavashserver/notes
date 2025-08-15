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

```typescript
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

```typescript
interface User {
  id: number;
  name: string;
  isAdmin?: boolean; // optional
}

const u: User = { id: 1, name: "Alice" };
```

```typescript
interface Admin extends User {
  permissions: string[];
}
```

```typescript
interface Greeter {
  (name: string): string;
}
```

### Pitfalls

- Interfaces can’t represent some advanced type constructs as well as type
  aliases (`type`), but they are better for object shapes and class contracts.

## Generics

Generics let you create **reusable, type-safe components** that work with
multiple types without losing type information.

| `any`                    | Generics                     |
| ------------------------ | ---------------------------- |
| Loses type information   | Preserves type relationships |
| No compile-time checking | Enforces type constraints    |
| Less readable contracts  | Self-documenting             |

```typescript
function identity<T>(value: T): T {
  return value;
}
```

### Generic Functions

```typescript
function identity<T>(value: T): T {
  return value;
}

let a = identity<string>("Hello"); // string
let b = identity(42); // inferred as number
```

### Generic Interfaces

```typescript
interface ApiResponse<T> {
  data: T;
  status: number;
}

const userResponse: ApiResponse<{ id: number; name: string }> = {
  data: { id: 1, name: "Alice" },
  status: 200,
};
```

### Generic Classes

```typescript
class Storage<T> {
  private items: T[] = [];
  add(item: T) {
    this.items.push(item);
  }
  getAll(): T[] {
    return this.items;
  }
}

const numStore = new Storage<number>();
numStore.add(1);
```

### Generic Constraints

Restrict what types can be used with a generic.

```typescript
interface HasId {
  id: number;
}

function printId<T extends HasId>(obj: T) {
  console.log(obj.id);
}

printId({ id: 5, name: "Test" }); // ✅
```

### Default Type Parameters

Provide a fallback type if none is given.

```typescript
function log<T = string>(value: T): void {
  console.log(value);
}

log(); // defaults to string
```

### Multiple Type Parameters

```typescript
function merge<A, B>(a: A, b: B): A & B {
  return { ...a, ...b };
}

const merged = merge({ name: "Alice" }, { age: 30 });
// merged: { name: string; age: number }
```

### Generics with `keyof`

Work with specific keys of an object type.

```typescript
function getProp<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { id: 1, name: "Bob" };
getProp(user, "name"); // string
```

### Generics with Conditional Types

```typescript
type ElementType<T> = T extends (infer U)[] ? U : T;
type A = ElementType<string[]>; // string
type B = ElementType<number>; // number
```

### Distributive Conditional Generics

```typescript
type ExcludeType<T, U> = T extends U ? never : T;
type Result = ExcludeType<string | number, number>; // string
```

### Generic Factory Functions

```typescript
function create<T>(c: { new (): T }): T {
  return new c();
}
```

### Generic Singleton Pattern

```typescript
class Singleton<T> {
  private static instance: any;
  private constructor() {}
  static getInstance<T>(cls: new () => T): T {
    if (!this.instance) this.instance = new cls();
    return this.instance;
  }
}
```

## Enums

Represent a set of **named constants**. Improves readability, enforces valid
values, and reduces magic strings/numbers.

```typescript
enum Direction {
  Up,
  Down,
  Left,
  Right,
}
```

- **Numeric enums** auto-increment: `Up = 0, Down = 1, ...`
- **String enums** must have explicit values:

```typescript
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

```typescript
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

```typescript
class Test {
  #secret = 123;
}
```

## Type Inference

Let TS deduce the type without explicit annotation. Less boilerplate, still
type-safe.

```typescript
let x = 42; // inferred as number
const y = "abc"; // inferred as string literal type
```

In function parameters, TS never infers from usage — you must annotate if you
want a type:

```typescript
function greet(name) {} // name = any, unless noImplicitAny = true
```

## Union & Intersection Types

Union is for _either/or_, Intersection is for _both_.

**Union (`|`)** — value can be one of several types.
**Intersection (`&`)** — combine multiple types into one.

```typescript
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

```typescript
const el = document.getElementById("app")!;
el.innerHTML = "Loaded!";
```

## Literal Types

Restrict variable to specific literal values. Good for finite config options or
state machine states.

```typescript
let role: "admin" | "user";
role = "admin"; // ✅
role = "guest"; // ❌
```

## Conditional Types

A **type-level if-else** — lets you create a type based on a condition.

- If `T` is assignable to `U`, result is `X`, else `Y`.

```typescript
T extends U ? X : Y
```

### Basic Example

```typescript
type IsString<T> = T extends string ? "Yes" : "No";

type A = IsString<string>; // "Yes"
type B = IsString<number>; // "No"
```

### With Generics

```typescript
function processValue<T>(value: T): T extends string ? string[] : T[] {
  return (typeof value === "string" ? value.split("") : [value]) as any;
}

const result1 = processValue("hi"); // string[]
const result2 = processValue(123); // number[]
```

### Key Use Cases

#### Type transformations

```typescript
type ElementType<T> = T extends (infer U)[] ? U : T;
type E1 = ElementType<string[]>; // string
type E2 = ElementType<number>; // number
```

#### Excluding or extracting

```typescript
type NonString<T> = T extends string ? never : T;
type Filtered = NonString<string | number | boolean>; // number | boolean
```

#### Function overload behavior

```typescript
type ReturnTypeIfString<T> = T extends string ? number : boolean;
let example: ReturnTypeIfString<"abc">; // number
```

### Pitfalls

- Conditional types **distribute** over unions:

```typescript
type Dist<T> = T extends string ? "yes" : "no";
type D = Dist<string | number>; // "yes" | "no"
```

- Use parentheses to avoid:

```typescript
type NoDist<T> = [T] extends [string] ? "yes" : "no";
type ND = NoDist<string | number>; // "no"
```

## Mapped Types

Create new types by transforming **each property** in an existing type.

```typescript
{ [K in Keys]: NewType }
```

### Basic Example

```typescript
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

```typescript
type Optional<T> = { [K in keyof T]?: T[K] };
type Readonly<T> = { readonly [K in keyof T]: T[K] };
type Mutable<T> = { -readonly [K in keyof T]: T[K] };
```

### With `keyof`

```typescript
type Keys = keyof User; // "id" | "name"

type Nullable<T> = { [K in keyof T]: T[K] | null };
type NullableUser = Nullable<User>;
// { id: number | null; name: string | null }
```

### Key Remapping (`as`) _(TS 4.1+)_

```typescript
type RenameKeys<T> = {
  [K in keyof T as `prefix_${string & K}`]: T[K];
};

type PrefixedUser = RenameKeys<User>;
// { prefix_id: number; prefix_name: string }
```

### Common Use Cases

#### Utility Types Implementation

- `Partial<T>`:

  ```typescript
  type Partial<T> = { [K in keyof T]?: T[K] };
  ```

- `Omit<T, K>`:

  ```typescript
  type Omit<T, K extends keyof any> = Pick<T, Exclude<keyof T, K>>;
  ```

#### Deep transformations

```typescript
type DeepPartial<T> = {
  [K in keyof T]?: T[K] extends object ? DeepPartial<T[K]> : T[K];
};
```

#### Dynamic API responses

```typescript
type ApiResponse<T> = {
  [K in keyof T]: { value: T[K]; loaded: boolean };
};
```

## Combining Conditional + Mapped Types

### Make properties optional if they are functions

```typescript
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

```typescript
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

## `tsc` (TypeScript Compiler)

`tsc` is the official TypeScript compiler CLI tool.

### Responsibilities

- Reads `.ts` (TypeScript) files and converts them to `.js` (JavaScript) files.
- Optionally performs **type-checking** (can even do it without emitting JS if
  needed).
- Uses **`tsconfig.json`** to determine compilation rules.

## `tsconfig.json`

A **configuration file** that tells `tsc` **how** to compile TypeScript code.

### Main Purposes

- Specifies which files to include/exclude.
- Defines target JavaScript version (`ES2017`, `ES2020`, etc.).
- Enables/disables compiler features (strict mode, decorators, etc.).
- Sets up path aliases for cleaner imports.
- Controls module format (`CommonJS`, `ESNext`, etc.).

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "strict": true,
    "outDir": "./dist",
    "baseUrl": "./src",
    "paths": {
      "@app/*": ["app/*"]
    }
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Angular Build Process & Where They Fit

When you run:

```bash
ng build
```

Angular **does NOT** simply run `tsc` directly. Instead, Angular uses
**`@angular-devkit/build-angular`** (Webpack-based builder) + **`ngc`** (Angular
Compiler, a superset of `tsc`).

### The flow

1. **Angular CLI loads `angular.json`** – determines build settings
   (optimization, environment).
2. **Loads `tsconfig.app.json`** (and `tsconfig.base.json`) – passes compiler
   options to `ngc`.
3. **`ngc` (Angular Compiler)**:

   - Runs **type-checking** (like `tsc`).
   - Performs **Ahead-of-Time (AOT)** compilation (turns Angular HTML templates
     into TypeScript).

4. **Webpack**:

   - Bundles the generated JS into optimized chunks.
   - Applies loaders for CSS, HTML, assets.

### Angular's Multiple `tsconfig` files

- `tsconfig.base.json` → Base config for the whole workspace.
- `tsconfig.app.json` → For application source code.
- `tsconfig.spec.json` → For unit tests.
- `tsconfig.json` → Sometimes just extends the base.

---

## Utility types

### `Partial<T>`

Make all properties in a type **optional**. Useful when updating objects — you
might only pass some fields.

```typescript
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

```typescript
interface User {
  id?: number;
  name?: string;
}

type StrictUser = Required<User>;
```

### `Readonly<T>`

Make all properties **immutable**. Helps enforce immutability in functional
programming or Redux state.

```typescript
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

```typescript
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

```typescript
type UserWithoutEmail = Omit<User, "email">;
```

### `Record<K, T>`

Create an object type with keys `K` and values `T`. Useful for dictionary-like
structures.

```typescript
type Roles = "admin" | "user";
type RolePermissions = Record<Roles, string[]>;

const perms: RolePermissions = {
  admin: ["read", "write"],
  user: ["read"],
};
```

### `Exclude<T, U>`

Remove from `T` all types assignable to `U`. Useful for filtering unions.

```typescript
type Status = "active" | "inactive" | "deleted";
type ActiveStatus = Exclude<Status, "deleted">; // "active" | "inactive"
```

### `Extract<T, U>`

Keep only the types from `T` assignable to `U`.

```typescript
type Status = "active" | "inactive" | "deleted";
type RemovedStatus = Extract<Status, "deleted">; // "deleted"
```

### `NonNullable<T>`

Remove `null` and `undefined` from a type.

```typescript
type MaybeString = string | null | undefined;
type DefiniteString = NonNullable<MaybeString>; // string
```

---

## Interview Questions

### When would you use `unknown` instead of `any`, and what’s the difference?

- **`any`** disables type checking completely — you can perform any operation
  without compiler errors.
- **`unknown`** is a safer alternative — you must first **narrow the type**
  before using it.

```typescript
let value: any = 5;
value.trim(); // OK at compile time, runtime error if not a string

let val: unknown = "Hello";
val.trim(); // ❌ Compile-time error
if (typeof val === "string") {
  val.trim(); // ✅ OK
}
```

### If an interface extends a class, what does it inherit?

It inherits **only the class's shape**, not its implementation. It also
**inherits private/protected members**, meaning only subclasses can implement
it.

```typescript
class Base {
  protected id!: number;
}

interface MyInterface extends Base {
  name: string;
}

class MyClass extends Base implements MyInterface {
  name = "Test";
}
```

### What are `const enums` and when to use them?

`const enum` values are **inlined** at compile time for performance — no object
is generated.

```typescript
const enum Directions {
  Up,
  Down,
}
let move = Directions.Up; // compiled to: let move = 0;
```

### Explain variance in TypeScript (covariance, contravariance)

- **Covariant**: Can substitute a subtype where a supertype is expected (e.g.,
  return types).
- **Contravariant**: Function parameters accept supertypes of declared type.

```typescript
type Animal = { name: string };
type Dog = { name: string; breed: string };

let dogFn: (arg: Dog) => void;
let animalFn: (arg: Animal) => void;

dogFn = animalFn; // OK (contravariance on parameters is unsafe in strict mode)
```

### How does `keyof` behave with index signatures?

```typescript
type MyType = { [key: string]: number; a: number };
type Keys = keyof MyType; // string | number
```

- With string index signatures, `keyof` produces `string | number` (since JS
  object keys can be strings or symbols, and numeric keys are coerced to
  strings).

### `infer` keyword usage

Used inside conditional types to extract a type.

```typescript
type ReturnTypeOf<T> = T extends (...args: any[]) => infer R ? R : never;

type Foo = () => number;
type Result = ReturnTypeOf<Foo>; // number
```

### TypeScript Narrowing Pitfall

```typescript
function process(val: string | number) {
  if (typeof val === "string" && val.length > 5) {
    // val is string here ✅
  } else {
    // val is still string | number ❌
  }
}
```

- Outside the `if`, TypeScript loses the narrowing.

### Exhaustive Checking with `never`

Ensures you know how to enforce type safety in unions.

```typescript
type Shape = { kind: "circle" } | { kind: "square" };

function area(shape: Shape) {
  switch (shape.kind) {
    case "circle":
      return 1;
    case "square":
      return 2;
    default:
      const _exhaustive: never = shape; // compile error if new case added
  }
}
```
