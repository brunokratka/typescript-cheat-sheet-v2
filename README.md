# TypeScript Language Reference & Cheat Sheet: Simple to Advanced (v2 Complete Edition)

This cheat sheet provides a comprehensive, concept-by-concept guide structured top-to-bottom according to the official TypeScript Handbook navigation. Every topic includes a clear conceptual breakdown, a **Simple Example** (foundational syntax), an **Advanced Example** (real-world pattern or complex type manipulation), and a dedicated **Tips & Hidden Details** block uncovering compiler flags, edge cases, type system behaviors, and performance gotchas from the official documentation.

---

## 1. Handbook Core

### The Basics
TypeScript adds a static type system on top of JavaScript to catch bugs (like typos, uncalled functions, or logic errors) at compile time before execution. During compilation, type annotations are completely erased, producing clean JavaScript that runs anywhere. Downleveling allows modern ECMAScript features (like ES2015 template literals) to compile down to older target environments like ES3 or ES5.

* **Simple Example:**
  ```ts
  // Explicit primitive type annotations
  let message: string = "Hello World";
  let count: number = 42;

  function greet(person: string, date: Date): string {
    return `Hello ${person}, today is ${date.toDateString()}!`;
  }
  ```

* **Advanced Example (Strictness Flags & Downleveling):**
  ```ts
  // Compiler behavior under strictNullChecks and noImplicitAny:
  function processUser(id: number, name?: string | null): string {
    // strictNullChecks forces handling null/undefined before accessing properties
    if (name == null) {
      return `User #${id}: Anonymous`;
    }
    return `User #${id}: ${name.trim().toUpperCase()}`;
  }
  ```

* **Tips & Hidden Details:**
  * **Erased Types:** Types exist solely for compile-time checking. They leave no runtime footprint or performance overhead.
  * **Emitting with Errors:** By default, `tsc` emits JavaScript output even if there are type errors (so your code still runs). Use the `--noEmitOnError` flag in `tsconfig.json` to block JS output when compilation fails.
  * **The Billion-Dollar Mistake:** Enabling `strictNullChecks` stops `null` and `undefined` from being implicitly assigned to every other type, catching widespread runtime crashes.
  * **Non-Exception Failures:** TypeScript catches errors that standard JS permits without throwing exceptions (e.g., calling un-callable objects, misspelling property names like `toLocaleLowercase()`).

---

### Everyday Types
Everyday types form the core primitive and composite building blocks in TypeScript, including primitives (`string`, `number`, `boolean`, `bigint`, `symbol`), arrays (`T[]` or `Array<T>`), `any`, union types (`A | B`), type aliases (`type`), interfaces (`interface`), type assertions (`as T`), non-null assertions (`!`), and const assertions (`as const`).

* **Simple Example:**
  ```ts
  // Type alias vs Interface definition
  type PointAlias = { x: number; y: number };
  interface PointInterface { x: number; y: number; }

  // Union types and literal values
  type Status = "pending" | "success" | "error";
  let currentStatus: Status = "pending";
  ```

* **Advanced Example (Literal Inference & Immutable Const Assertions):**
  ```ts
  // Deep immutability and exact literal inference with `as const`
  const config = {
    endpoint: "https://api.example.com",
    retries: 3,
    methods: ["GET", "POST"]
  } as const;

  // Derived type: { readonly endpoint: "https://api.example.com"; readonly retries: 3; readonly methods: readonly ["GET", "POST"]; }
  type ApiConfig = typeof config;
  ```

* **Tips & Hidden Details:**
  * **Type Aliases vs. Interfaces:** Interfaces can be reopened and merged via Declaration Merging across files, whereas Type Aliases cannot be re-declared.
  * **Non-Null Assertion (`!`):** Writing `val!` tells the compiler "I guarantee this isn't `null` or `undefined`." It is erased at runtime and offers no runtime safety if wrong.
  * **Type Assertions (`as T`):** Only allows assertions to more specific or less specific versions of a type. Impossible conversions (e.g., `string as number`) require bridging through `unknown` (`expr as unknown as number`).
  * **JavaScript `typeof` Gotcha:** `typeof null === "object"`. In TypeScript, checking `typeof x === "object"` does not remove `null` from a union type automatically without explicit null checks.

---

### Narrowing
Narrowing is the process where TypeScript refines broad types (like unions) into narrower, specific types within code execution branches using Control Flow Analysis (CFA), type guards (`typeof`, `instanceof`, `in`), equality checks, truthiness evaluation, user-defined type predicates (`x is T`), assertion functions, and discriminated unions.

* **Simple Example:**
  ```ts
  function padLeft(padding: number | string, input: string): string {
    if (typeof padding === "number") {
      return " ".repeat(padding) + input; // padding is number
    }
    return padding + input; // padding is string
  }
  ```

* **Advanced Example (Discriminated Unions, User-Defined Predicates & Exhaustiveness):**
  ```ts
  interface Circle { kind: "circle"; radius: number; }
  interface Square { kind: "square"; sideLength: number; }
  type Shape = Circle | Square;

  // Custom User-Defined Type Predicate
  function isCircle(shape: Shape): shape is Circle {
    return shape.kind === "circle";
  }

  function getArea(shape: Shape): number {
    if (isCircle(shape)) {
      return Math.PI * shape.radius ** 2; // shape is Circle
    }
    switch (shape.kind) {
      case "square":
        return shape.sideLength ** 2;
      default:
        // Exhaustiveness checking with never: triggers compile error if new Shape added
        const _exhaustiveCheck: never = shape;
        return _exhaustiveCheck;
    }
  }
  ```

* **Tips & Hidden Details:**
  * **`== null` Loose Equality Trick:** Checking `val == null` or `val != null` handles **both** `null` and `undefined` in a single check.
  * **The `in` Operator & Optional Properties:** Checking `"prop" in obj` narrows types. However, optional properties (`prop?: string`) will appear in both `true` and `false` branches of `in` checks.
  * **Control Flow Assignment:** Variable types are tracked dynamically as values are assigned along execution paths.
  * **Assertion Functions:** Functions marked `asserts condition` or `asserts val is T` throw runtime errors if false and narrow the caller's scope for subsequent lines without returning a boolean.

---

### More on Functions
Functions in TypeScript support parameter and return annotations, optional parameters (`?`), rest parameters and rest arguments, function overloads, custom `this` type declarations, call signatures, construct signatures, and special return types (`void`, `unknown`, `never`, `object`).

* **Simple Example:**
  ```ts
  // Function Type Expression & Optional Parameter
  type GreetFn = (name: string, title?: string) => string;

  const greetUser: GreetFn = (name, title) => {
    return title ? `Hello, ${title} ${name}` : `Hello, ${name}`;
  };
  ```

* **Advanced Example (Function Overloads, Call/Construct Signatures & Explicit `this`):**
  ```ts
  // Overload Signatures (visible to callers)
  function makeDate(timestamp: number): Date;
  function makeDate(m: number, d: number, y: number): Date;
  // Implementation Signature (hidden from callers!)
  function makeDate(mOrTimestamp: number, d?: number, y?: number): Date {
    if (d !== undefined && y !== undefined) {
      return new Date(y, mOrTimestamp - 1, d);
    }
    return new Date(mOrTimestamp);
  }

  // Callable & Constructible Object Type with explicit 'this'
  interface ComponentFactory {
    (name: string): HTMLElement; // Call signature
    new (name: string): HTMLElement; // Construct signature
    description: string; // Property on function
  }
  ```

* **Tips & Hidden Details:**
  * **The `void` Return Assignability Rule:** A contextual function type with a `void` return (`() => void`) **allows returning non-void values**, but callers cannot use the return value. Only literal function declarations returning `void` forbid `return value;`.
  * **Implementation Signature Invisibility:** The implementation signature of an overloaded function cannot be called directly; only overload signatures are visible to external consumers.
  * **Optional Parameters in Callbacks Pitfall:** Avoid optional parameters in callback declarations (e.g., `(arr: any[], index?: number) => void`), because callers might pass callbacks that expect strict argument counts.
  * **Rest Arguments Spreading:** Spreading non-tuple arrays into function calls requires type assertions or `as const` tuples so TypeScript knows the exact parameter length.

---

### Object Types
Object types define data structures using properties, optional flags (`?`), `readonly` modifiers, index signatures (`[key: string]: T`), interface extension (`extends`), intersection types (`&`), and tuple types (including optional, rest, and `readonly` tuples).

* **Simple Example:**
  ```ts
  interface UserProfile {
    readonly id: string;
    name: string;
    email?: string; // Optional property
  }

  // Interface extension
  interface AdminProfile extends UserProfile {
    permissions: string[];
  }
  ```

* **Advanced Example (Index Signatures, Excess Checks & Tuples with Rest Elements):**
  ```ts
  // Dictionary index signature accepting string keys
  interface FlexibleCache {
    [key: string]: string | number;
    defaultTTL: number; // Must conform to index signature return type!
  }

  // Tuple with leading elements and variable rest elements
  type HttpHeader = [statusCode: number, statusText: string, ...headers: string[]];
  type ReadonlyPoint = readonly [x: number, y: number, z?: number];
  ```

* **Tips & Hidden Details:**
  * **Shallow Immutability of `readonly`:** Marking a property `readonly` prevents reassigning the property reference itself (`obj.prop = newVal`), but internal nested properties of an object or array can still be mutated.
  * **Numeric Index Subtype Rule:** In index signatures, a `number` index signature **must return a subtype** of the `string` index signature, because JS coerces numeric object keys to strings at runtime (`obj[0]` is identical to `obj["0"]`).
  * **Excess Property Checks:** Object literals passed directly to functions trigger excess property checks, flagging unknown properties as errors. Assigning the object literal to an intermediate variable first bypasses excess property checks!
  * **Intersection vs. Extension Collisions:** Extending interfaces with conflicting property types throws a compile error at declaration time. Intersecting conflicting types (`type C = A & B`) compiles cleanly but resolves the conflicting property to `never`.

---

## 2. Type Manipulation

### Creating Types from Types
TypeScript allows expressing new types in terms of existing types or runtime values by combining generic parameters with type operators: `keyof`, `typeof`, indexed access, conditional types, mapped types, and template literal types.

* **Simple Example:**
  ```ts
  type Point = { x: number; y: number };
  // Creating a key union type from an existing object type
  type PointKeys = keyof Point; // "x" | "y"
  ```

* **Advanced Example (Deriving Types from Runtime Values):**
  ```ts
  const appConfig = { env: "production", port: 8080, debug: false };

  // Query value type, extract keys, and construct a mapped nullable type
  type AppConfig = typeof appConfig;
  type ConfigKey = keyof AppConfig;
  type NullableConfig = { [K in ConfigKey]: AppConfig[K] | null };
  ```

* **Tips & Hidden Details:**
  * Type creation operators can be chained infinitely to derive complex, self-updating domain models that adapt automatically whenever underlying data structures change.

---

### Generics
Generics allow building reusable, type-safe components that operate over multiple data types using type parameters (`<T>`), generic constraints (`extends`), `keyof` constraints, generic defaults, and generic class constructors.

* **Simple Example:**
  ```ts
  function identity<T>(arg: T): T {
    return arg;
  }

  let output = identity<string>("myString");
  ```

* **Advanced Example (Generic Constraints, `keyof` & Class Constructor Factories):**
  ```ts
  // Enforcing that Key must exist on Object Type
  function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
    return obj[key];
  }

  // Factory function requiring a construct signature
  function createInstance<T>(ctor: new () => T): T {
    return new ctor();
  }
  ```

* **Tips & Hidden Details:**
  * **Push Type Parameters Down:** Prefer using the type parameter directly in parameters rather than constraining it needlessly; let inference do the work.
  * **Static Members Limitation:** In generic classes, `static` members **cannot reference class type parameters** because static members belong to the single runtime constructor object, not individual generic instances.

---

### Keyof Type Operator
The `keyof` operator takes an object type and produces a string or numeric literal union of its keys. When applied to types with index signatures, it returns `string | number`.

* **Simple Example:**
  ```ts
  interface Person { name: string; age: number; }
  type PersonKeys = keyof Person; // "name" | "age"
  ```

* **Advanced Example (Index Signature Key Extraction):**
  ```ts
  interface StringArray { [index: number]: string; }
  type ArrayKeys = keyof StringArray; // number

  interface Dict { [key: string]: boolean; }
  type DictKeys = keyof Dict; // string | number (because JS coerces numbers to strings)
  ```

* **Tips & Hidden Details:**
  * `keyof Dict` on a string index signature yields `string | number` because JavaScript automatically coerces numeric property keys to strings when indexing objects (`obj[10]` becomes `obj["10"]`).

---

### Typeof Type Operator
TypeScript extends JavaScript's runtime `typeof` operator by allowing it to operate inside **type contexts** to query the compile-time type of an identifier or property.

* **Simple Example:**
  ```ts
  let s = "hello";
  type StringType = typeof s; // string
  ```

* **Advanced Example (Extracting Function & Value Shapes):**
  ```ts
  function calculateTotal(price: number, tax: number) {
    return price + price * tax;
  }

  // Extract function signature type from variable identifier
  type MathFn = typeof calculateTotal; // (price: number, tax: number) => number
  ```

* **Tips & Hidden Details:**
  * **Strict Limitation:** `typeof` in a type context **can only be executed on identifiers or property accesses** (e.g., `typeof myVar` or `typeof obj.prop`). You **cannot** use `typeof` on function call expressions (e.g., `typeof msg()` is illegal).

---

### Indexed Access Types
Indexed access types (`Type[Key]`) look up specific property types on another type using brackets. Indexing with unions or `keyof` extracts property type unions, while indexing with `number` extracts array or tuple element types.

* **Simple Example:**
  ```ts
  type User = { id: number; name: string; active: boolean };
  type UserId = User["id"]; // number
  ```

* **Advanced Example (Extracting Union Properties & Array Elements):**
  ```ts
  const MyArray = [
    { name: "Alice", age: 15 },
    { name: "Bob", age: 23 },
  ];

  // Lookup property type union across array elements
  type Person = typeof MyArray[number]; // { name: string; age: number }
  type AgeType = typeof MyArray[number]["age"]; // number
  ```

* **Tips & Hidden Details:**
  * **No Variable Indexing:** You must use type references or type aliases inside index access brackets (`Type[MyTypeKey]`). Referencing runtime variable names inside `Type[keyVar]` throws a compiler error.

---

### Conditional Types
Conditional types (`T extends U ? X : Y`) select output types based on structural type assignment checks, supporting type constraints, element extraction with `infer`, and automatic distribution across union types.

* **Simple Example:**
  ```ts
  type IsString<T> = T extends string ? true : false;
  type Test1 = IsString<"hello">; // true
  type Test2 = IsString<42>; // false
  ```

* **Advanced Example (`infer` Keyword & Distributive Distro Prevention):**
  ```ts
  // Extract return type of function type using infer
  type GetReturnType<T> = T extends (...args: any[]) => infer R ? R : never;

  // Naked type parameters distribute across unions:
  type ToArray<T> = T extends any ? T[] : never;
  type StrOrNumArray = ToArray<string | number>; // string[] | number[]

  // Wrapping type parameters in tuples PREVENTS distribution:
  type ToNonDistributiveArray<T> = [T] extends [any] ? T[] : never;
  type SingleUnionArray = ToNonDistributiveArray<string | number>; // (string | number)[]
  ```

* **Tips & Hidden Details:**
  * **Distributive Semantics:** When a conditional type acts on a naked generic parameter `T`, passing a union type `A | B` causes it to distribute: `(A extends U ? X : Y) | (B extends U ? X : Y)`. Wrap `[T]` in a tuple to evaluate the union as a whole.

---

### Mapped Types
Mapped types build on index signature syntax to iterate through keys (usually produced via `keyof`) to transform property types, add or remove modifiers (`readonly`, `?`), or remap property names using the `as` clause.

* **Simple Example:**
  ```ts
  type OptionsFlags<T> = {
    [Property in keyof T]: boolean;
  };

  type FeatureFlags = OptionsFlags<{ darkMode: () => void; beta: () => void }>;
  // Result: { darkMode: boolean; beta: boolean }
  ```

* **Advanced Example (Mapping Modifiers & Key Remapping with `as` Filtering):**
  ```ts
  // Strip readonly and optional modifiers using - prefix
  type ConcreteMutable<T> = {
    -readonly [P in keyof T]-?: T[P];
  };

  // Remap keys via template literals and filter out specific fields with never
  type Getters<T> = {
    [K in keyof T as K extends "id" ? never : `get${Capitalize<string & K>}`]: () => T[K];
  };

  interface User { id: number; name: string; age: number; }
  type UserGetters = Getters<User>; // { getName: () => string; getAge: () => number }
  ```

* **Tips & Hidden Details:**
  * **Key Filtering with `never`:** Producing `never` as a remapped key name in an `as` clause completely excludes that property from the resulting mapped type object.
  * **Modifier Prefixes:** Using `-readonly` or `-?` explicitly removes immutable or optional constraints, while `+readonly` or `+?` adds them.

---

### Template Literal Types
Template literal types build string types by interpolating union types into backtick string templates, triggering automatic cross-product expansion, string pattern type inference with `infer`, and intrinsic letter-case transformations.

* **Simple Example:**
  ```ts
  type World = "world";
  type Greeting = `hello ${World}`; // "hello world"
  ```

* **Advanced Example (Cross-Product Union Expansion & Pattern Inference):**
  ```ts
  type VerticalAlignment = "top" | "bottom";
  type HorizontalAlignment = "left" | "right";
  // Generates cross-product union of all combinations!
  type Alignment = `${VerticalAlignment}-${HorizontalAlignment}`; 
  // "top-left" | "top-right" | "bottom-left" | "bottom-right"

  // Extract ID from formatted string type
  type ExtractId<S> = S extends `item_${infer ID}` ? ID : never;
  type ItemId = ExtractId<"item_12345">; // "12345"
  ```

* **Tips & Hidden Details:**
  * **Intrinsic String Manipulation Helpers:** TypeScript provides built-in type helpers for string case adjustments: `Uppercase<S>`, `Lowercase<S>`, `Capitalize<S>`, and `Uncapitalize<S>`.

---

## 3. Object-Oriented & Modular TypeScript

### Classes
Classes in TypeScript add static type annotations, member visibility modifiers (`public`, `protected`, `private`), parameter properties, getters/setters, static blocks, abstract classes, polymorphic `this` types, and `this`-based type guards to ES6 classes.

* **Simple Example:**
  ```ts
  class Point {
    x: number;
    y: number;

    constructor(x: number, y: number = 0) {
      this.x = x;
      this.y = y;
    }
  }
  ```

* **Advanced Example (Parameter Properties, Soft/Hard Private, `this` Type Guards & Abstract Classes):**
  ```ts
  abstract class BaseEntity {
    // Parameter properties automatically create and initialize fields
    constructor(public readonly id: string, protected createdAt: Date) {}
    abstract validate(): boolean;
  }

  class UserAccount extends BaseEntity {
    private tsPrivateField = "soft private"; // Accessible via obj['tsPrivateField'] at runtime
    #jsHardPrivateField = "hard private"; // Enforced natively at runtime (compiled with WeakMaps in ES5)

    // Setter without explicit getter type
    set password(pass: string) { /* ... */ }

    validate(): boolean { return true; }

    // Polymorphic 'this' type guard for chaining
    isUserAccount(): this is UserAccount {
      return this instanceof UserAccount;
    }
  }
  ```

* **Tips & Hidden Details:**
  * **TS `private` vs. JS `#private`:** TypeScript's `private` keyword is purely compile-time; members can still be accessed at runtime using bracket notation (`user["tsPrivateField"]`). JavaScript `#field` syntax is hard-private and enforced natively at runtime.
  * **Getter/Setter Rules:** If a getter exists but no setter, the property is automatically inferred as `readonly`.
  * **Structural Class Gotcha:** TypeScript uses structural typing for classes. An empty class `class Empty {}` has no members, so **any object** is assignable to `Empty`!
  * **`implements` Clause Limitation:** `implements` checks that a class satisfies an interface, but it does **not** automatically infer or type method parameters inside the class implementation.

---

### Modules
Modules in TypeScript isolate scope across files using ES Module syntax (`import`/`export`), type-only imports (`import type`), CommonJS compatibility (`export =`), and compiler resolution settings (`moduleResolution`).

* **Simple Example:**
  ```ts
  // @filename: math.ts
  export function add(a: number, b: number): number { return a + b; }

  // @filename: main.ts
  import { add } from "./math.js";
  ```

* **Advanced Example (Type-Only Imports & CommonJS Interop):**
  ```ts
  // Explicit type-only imports prevent transpilers (Babel/swc/esbuild) from emitting empty imports
  import type { UserProfile } from "./types.js";
  import { getUser, type UserConfig } from "./api.js";

  // Legacy CommonJS module export mapping
  interface CommonJSExport { version: string; }
  export = { version: "1.0.0" };
  ```

* **Tips & Hidden Details:**
  * **Module vs. Script Scope:** Any file containing top-level `import` or `export` statements is executed as a module. Files without them are treated as global scripts. Add `export {};` to force a file to become a module.
  * **Inline Type Imports (TS 4.5+):** Allows mixing value and type imports cleanly: `import { value, type Type } from "./module"`.

---

## 4. Reference & Specialized Tools

### Utility Types
Utility types are globally available built-in generic helpers that transform existing type shapes, manipulate function/constructor signatures, or modify string literal cases.

* **Simple Example:**
  ```ts
  interface Todo { title: string; description: string; }
  type PartialTodo = Partial<Todo>; // All fields optional
  type ReadonlyTodo = Readonly<Todo>; // All fields immutable
  ```

* **Advanced Example (Comprehensive Utility Suite & Async Extraction):**
  ```ts
  interface Task { id: number; title: string; completed: boolean; tags: string[]; }

  type TaskPreview = Pick<Task, "id" | "title">;
  type TaskDraft = Omit<Task, "id">;
  type TaskMap = Record<string, Task>;

  type Union = "a" | "b" | "c" | null | undefined;
  type CleanUnion = NonNullable<Exclude<Union, "c">>; // "a" | "b"

  // Function Utilities
  function fetchTask(id: number): Promise<Task> { throw "impl"; }
  type FetchParams = Parameters<typeof fetchTask>; // [id: number]
  type FetchReturn = ReturnType<typeof fetchTask>; // Promise<Task>
  type UnwrappedTask = Awaited<ReturnType<typeof fetchTask>>; // Task
  ```

* **Tips & Hidden Details:**
  * **`Awaited<T>`:** Recursively unwraps nested `Promise<Promise<T>>` chains to extract the ultimate inner resolved type.
  * **`ThisType<T>`:** Marker utility used to specify the explicit contextual `this` inside object literal methods (requires `--noImplicitThis`).

---

### Decorators
Decorators (`@decorator`) add metaprogramming annotations and runtime hooks to classes, methods, accessors, properties, and parameters. They require `"experimentalDecorators": true` in `tsconfig.json`.

* **Simple Example:**
  ```ts
  function sealed(target: Function) {
    Object.seal(target);
    Object.seal(target.prototype);
  }

  @sealed
  class Greeter {
    greeting: string = "Hello";
  }
  ```

* **Advanced Example (Decorator Factory, Method Descriptor Patching & Metadata):**
  ```ts
  import "reflect-metadata";

  // Decorator factory
  function log(prefix: string) {
    return function (target: any, propertyKey: string, descriptor: PropertyDescriptor) {
      const originalMethod = descriptor.value;
      descriptor.value = function (...args: any[]) {
        console.log(`[${prefix}] Calling ${propertyKey}`);
        return originalMethod.apply(this, args);
      };
    };
  }

  class ApiService {
    @log("API")
    fetchData(@required url: string) { return url; }
  }

  function required(target: Object, propertyKey: string | symbol, parameterIndex: number) {
    // Intercepts parameter metadata
  }
  ```

* **Tips & Hidden Details:**
  * **Evaluation vs. Execution Order:** Decorator expressions are evaluated top-to-bottom, but decorator wrapper functions are **executed bottom-to-top** (composed like function calls `f(g(x))`).
  * **Design-Time Metadata:** Enabling `"emitDecoratorMetadata": true` alongside `reflect-metadata` causes `tsc` to inject runtime design-time types (`design:type`, `design:paramtypes`, `design:returntype`).

---

### Declaration Merging
Declaration merging allows the compiler to automatically merge multiple separate declarations sharing the exact same identifier into a single unified entity.

* **Simple Example:**
  ```ts
  // Merging two interface declarations
  interface Box { height: number; }
  interface Box { width: number; }

  const box: Box = { height: 10, width: 20 };
  ```

* **Advanced Example (Interface Overloads & Namespace Merging with Classes/Functions/Enums):**
  ```ts
  // Interface overload merging with string literal prioritization
  interface Document {
    createElement(tagName: string): HTMLElement; // Lower priority
  }
  interface Document {
    createElement(tagName: "div"): HTMLDivElement; // Bubbled to top!
    createElement(tagName: "span"): HTMLSpanElement; // Bubbled to top!
  }

  // Merging Namespace with Class (Inner Class pattern)
  class Album { label: Album.AlbumLabel; }
  namespace Album {
    export class AlbumLabel {}
  }

  // Merging Namespace with Enum (Static Methods on Enum)
  enum Color { Red = 1, Green = 2 }
  namespace Color {
    export function mix(): number { return Color.Red + Color.Green; }
  }
  ```

* **Tips & Hidden Details:**
  * **Literal Overload Bubbling:** In merged interfaces with method overloads, signatures containing a **single string literal parameter** automatically bubble to the top of the overload list.
  * **Positioning Namespace Merges:** Namespace declarations must be placed **after** the class, function, or enum declaration they merge with.

---

### Enums
Enums define named sets of numeric or string constants. Enums are a rare runtime addition to JavaScript by TypeScript, compiled into real JavaScript mapping objects unless marked as `const enum`.

* **Simple Example:**
  ```ts
  enum Direction {
    Up = 1,
    Down,
    Left,
    Right,
  }
  let dir = Direction.Up; // 1
  ```

* **Advanced Example (Numeric Reverse Mappings, Union Enums & `as Const` Alternative):**
  ```ts
  enum LogLevel { ERROR = 0, WARN = 1, INFO = 2 }

  // Extract key union type from Enum object
  type LogLevelKey = keyof typeof LogLevel; // "ERROR" | "WARN" | "INFO"

  // Numeric reverse mapping in JS output: LogLevel[0] === "ERROR"
  let nameOfError = LogLevel[LogLevel.ERROR]; // "ERROR"

  // Modern idiomatic 'as const' object alternative:
  const ODirection = { Up: 0, Down: 1 } as const;
  type DirectionType = typeof ODirection[keyof typeof ODirection]; // 0 | 1
  ```

* **Tips & Hidden Details:**
  * **Reverse Mappings:** Numeric enums generate reverse value-to-name object mappings in compiled JavaScript (`Enum[Enum["A"] = 0] = "A"`). **String enums generate NO reverse mappings**.
  * **`const enum` Inlining:** `const enum` declarations are completely erased during compilation, with values inlined directly at call sites. However, ambient `const enum` in `.d.ts` files can break downleveling or isolated compilation tools (esbuild/swc).
  * **Objects vs. Enums:** Standard JS objects with `as const` are widely preferred in modern TypeScript over `enum` syntax to maintain strict alignment with standard ECMAScript.

---

### Iterators and Generators
Iterable objects contain a `[Symbol.iterator]` method, allowing elements to be traversed using `for..of` loops. TypeScript downlevels iterator loops according to compiler `target` settings.

* **Simple Example:**
  ```ts
  const list = [10, 20, 30];

  for (const value of list) {
    console.log(value); // 10, 20, 30
  }
  ```

* **Advanced Example (Custom Iterable Object & Generator Function):**
  ```ts
  // Custom iterable implementing [Symbol.iterator]
  class CustomSequence implements Iterable<number> {
    *[Symbol.iterator](): Iterator<number> {
      yield 1;
      yield 2;
      yield 3;
    }
  }

  const seq = new CustomSequence();
  for (const num of seq) {
    console.log(num); // 1, 2, 3
  }
  ```

* **Tips & Hidden Details:**
  * **`for..of` vs. `for..in`:** `for..of` inspects and retrieves **values** from iterable objects, whereas `for..in` retrieves **property keys** (including array indices as strings).
  * **Downleveling Target Behavior:** When targeting ES5, `tsc` emits helper code for `for..of` loops over arrays, but non-array iterables require `--downlevelIteration` to compile correctly to ES5.
