# TypeScript

TypeScript is a strongly typed superset of JavaScript that compiles to plain JavaScript. It adds optional static typing, classes, and interfaces to JavaScript, making it easier to build and maintain large-scale applications.

## Why TypeScript?

- **Type Safety**: Catch errors at compile time
- **Better IDE Support**: Enhanced autocomplete and refactoring
- **Modern JavaScript Features**: ES6+ features with backward compatibility
- **Improved Maintainability**: Self-documenting code
- **Large Ecosystem**: Works with existing JavaScript libraries

## Installation

```bash
# Install TypeScript globally
npm install -g typescript

# Check version
tsc --version

# Initialize TypeScript project
tsc --init
```

## Basic Types

```typescript
// Primitive types
let name: string = "John";
let age: number = 30;
let isActive: boolean = true;
let nothing: null = null;
let notDefined: undefined = undefined;

// Arrays
let numbers: number[] = [1, 2, 3];
let strings: Array<string> = ["a", "b", "c"];

// Tuple
let tuple: [string, number] = ["John", 30];

// Enum
enum Color {
  Red,
  Green,
  Blue
}
let color: Color = Color.Red;

// Any (avoid when possible)
let anything: any = "could be anything";

// Unknown (safer than any)
let uncertain: unknown = 4;
if (typeof uncertain === "number") {
  let num: number = uncertain;
}

// Void
function logMessage(message: string): void {
  console.log(message);
}

// Never (function never returns)
function throwError(message: string): never {
  throw new Error(message);
}
```

## Interfaces

```typescript
// Basic interface
interface User {
  id: number;
  name: string;
  email: string;
  age?: number; // Optional property
  readonly createdAt: Date; // Read-only property
}

const user: User = {
  id: 1,
  name: "John",
  email: "john@example.com",
  createdAt: new Date()
};

// Interface with methods
interface Calculator {
  add(a: number, b: number): number;
  subtract(a: number, b: number): number;
}

// Extending interfaces
interface Employee extends User {
  department: string;
  salary: number;
}

// Index signatures
interface StringMap {
  [key: string]: string;
}

const map: StringMap = {
  name: "John",
  city: "New York"
};
```

## Type Aliases

```typescript
// Basic type alias
type ID = string | number;

// Object type
type Point = {
  x: number;
  y: number;
};

// Union types
type Status = "pending" | "approved" | "rejected";

// Intersection types
type Admin = User & {
  permissions: string[];
};

// Function type
type MathOperation = (a: number, b: number) => number;

const add: MathOperation = (a, b) => a + b;
```

## Functions

```typescript
// Function with types
function greet(name: string): string {
  return `Hello, ${name}!`;
}

// Optional parameters
function buildName(firstName: string, lastName?: string): string {
  return lastName ? `${firstName} ${lastName}` : firstName;
}

// Default parameters
function power(base: number, exponent: number = 2): number {
  return Math.pow(base, exponent);
}

// Rest parameters
function sum(...numbers: number[]): number {
  return numbers.reduce((acc, num) => acc + num, 0);
}

// Function overloads
function format(value: string): string;
function format(value: number): string;
function format(value: string | number): string {
  return typeof value === "string" ? value : value.toString();
}

// Arrow functions
const multiply = (a: number, b: number): number => a * b;

// Generic functions
function identity<T>(arg: T): T {
  return arg;
}

const result = identity<string>("hello");
```

## Classes

```typescript
class Person {
  // Properties
  private id: number;
  public name: string;
  protected age: number;
  readonly birthDate: Date;

  // Constructor
  constructor(id: number, name: string, age: number, birthDate: Date) {
    this.id = id;
    this.name = name;
    this.age = age;
    this.birthDate = birthDate;
  }

  // Methods
  greet(): string {
    return `Hello, I'm ${this.name}`;
  }

  // Getter
  get info(): string {
    return `${this.name} (${this.age})`;
  }

  // Setter
  set updateAge(age: number) {
    if (age > 0) {
      this.age = age;
    }
  }

  // Static method
  static create(name: string): Person {
    return new Person(Date.now(), name, 0, new Date());
  }
}

// Inheritance
class Employee extends Person {
  constructor(
    id: number,
    name: string,
    age: number,
    birthDate: Date,
    public department: string
  ) {
    super(id, name, age, birthDate);
  }

  greet(): string {
    return `${super.greet()} from ${this.department}`;
  }
}

// Abstract class
abstract class Animal {
  abstract makeSound(): void;

  move(): void {
    console.log("Moving...");
  }
}

class Dog extends Animal {
  makeSound(): void {
    console.log("Woof!");
  }
}

// Implementing interfaces
interface Printable {
  print(): void;
}

class Document implements Printable {
  print(): void {
    console.log("Printing document...");
  }
}
```

## Generics

```typescript
// Generic function
function toArray<T>(item: T): T[] {
  return [item];
}

// Generic interface
interface Repository<T> {
  getById(id: number): T;
  getAll(): T[];
  save(item: T): void;
}

// Generic class
class DataStore<T> {
  private data: T[] = [];

  add(item: T): void {
    this.data.push(item);
  }

  get(index: number): T {
    return this.data[index];
  }
}

const numberStore = new DataStore<number>();
numberStore.add(1);

// Generic constraints
interface HasLength {
  length: number;
}

function logLength<T extends HasLength>(item: T): void {
  console.log(item.length);
}

// Multiple type parameters
function merge<T, U>(obj1: T, obj2: U): T & U {
  return { ...obj1, ...obj2 };
}
```

## Advanced Types

### Union Types

```typescript
function format(value: string | number): string {
  if (typeof value === "string") {
    return value.toUpperCase();
  }
  return value.toFixed(2);
}
```

### Intersection Types

```typescript
type Person = {
  name: string;
};

type Employee = {
  employeeId: number;
};

type Staff = Person & Employee;

const staff: Staff = {
  name: "John",
  employeeId: 123
};
```

### Type Guards

```typescript
function isString(value: unknown): value is string {
  return typeof value === "string";
}

function process(value: string | number) {
  if (isString(value)) {
    console.log(value.toUpperCase());
  } else {
    console.log(value.toFixed(2));
  }
}
```

### Mapped Types

```typescript
type Readonly<T> = {
  readonly [P in keyof T]: T[P];
};

type Partial<T> = {
  [P in keyof T]?: T[P];
};

interface User {
  name: string;
  age: number;
}

type ReadonlyUser = Readonly<User>;
type PartialUser = Partial<User>;
```

### Conditional Types

```typescript
type IsString<T> = T extends string ? true : false;

type A = IsString<string>; // true
type B = IsString<number>; // false
```

### Utility Types

```typescript
interface User {
  id: number;
  name: string;
  email: string;
  age: number;
}

// Pick - Select specific properties
type UserPreview = Pick<User, "id" | "name">;

// Omit - Exclude specific properties
type UserWithoutId = Omit<User, "id">;

// Partial - Make all properties optional
type PartialUser = Partial<User>;

// Required - Make all properties required
type RequiredUser = Required<PartialUser>;

// Record - Create object type with specific keys
type UserRoles = Record<string, string[]>;

// ReturnType - Extract return type of function
function getUser() {
  return { id: 1, name: "John" };
}
type User = ReturnType<typeof getUser>;
```

## Modules

```typescript
// Exporting
export interface User {
  id: number;
  name: string;
}

export class UserService {
  getUser(id: number): User {
    return { id, name: "John" };
  }
}

export default UserService;

// Importing
import UserService, { User } from "./userService";
import * as UserModule from "./userService";
```

## Decorators

```typescript
// Class decorator
function sealed(constructor: Function) {
  Object.seal(constructor);
  Object.seal(constructor.prototype);
}

@sealed
class BugReport {
  type = "report";
  title: string;

  constructor(t: string) {
    this.title = t;
  }
}

// Method decorator
function log(target: any, propertyKey: string, descriptor: PropertyDescriptor) {
  const originalMethod = descriptor.value;

  descriptor.value = function (...args: any[]) {
    console.log(`Calling ${propertyKey} with`, args);
    return originalMethod.apply(this, args);
  };

  return descriptor;
}

class Calculator {
  @log
  add(a: number, b: number): number {
    return a + b;
  }
}
```

## Async/Await

```typescript
async function fetchUser(id: number): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();
  return data;
}

// Error handling
async function getUserSafely(id: number): Promise<User | null> {
  try {
    const user = await fetchUser(id);
    return user;
  } catch (error) {
    console.error("Error fetching user:", error);
    return null;
  }
}
```

## Type Assertions

```typescript
// As syntax
let value: unknown = "hello";
let length: number = (value as string).length;

// Angle bracket syntax (not in JSX)
let length2: number = (<string>value).length;

// Non-null assertion
function getValue(): string | null {
  return "value";
}

let value2 = getValue()!; // Assert non-null
```

## tsconfig.json

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "lib": ["ES2020"],
    "outDir": "./dist",
    "rootDir": "./src",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "declaration": true,
    "declarationMap": true,
    "sourceMap": true,
    "noUnusedLocals": true,
    "noUnusedParameters": true,
    "noImplicitReturns": true,
    "noFallthroughCasesInSwitch": true
  },
  "include": ["src/**/*"],
  "exclude": ["node_modules", "dist"]
}
```

## Best Practices

1. **Enable Strict Mode**: Use `"strict": true` in tsconfig.json
2. **Avoid Any**: Use `unknown` or specific types instead
3. **Use Interfaces for Objects**: Prefer interfaces over type aliases for objects
4. **Leverage Type Inference**: Let TypeScript infer types when obvious
5. **Use Readonly**: Mark properties readonly when they shouldn't change
6. **Prefer Union Types**: Use union types over enums when possible
7. **Use Utility Types**: Leverage built-in utility types
8. **Type Guards**: Use type guards for runtime type checking
9. **Null Checks**: Enable `strictNullChecks`
10. **Consistent Naming**: Use PascalCase for types/interfaces, camelCase for variables

## Common Patterns

### Singleton Pattern

```typescript
class Database {
  private static instance: Database;

  private constructor() {}

  static getInstance(): Database {
    if (!Database.instance) {
      Database.instance = new Database();
    }
    return Database.instance;
  }
}
```

### Factory Pattern

```typescript
interface Product {
  name: string;
  price: number;
}

class ProductFactory {
  static create(type: string): Product {
    switch (type) {
      case "book":
        return { name: "Book", price: 10 };
      case "phone":
        return { name: "Phone", price: 500 };
      default:
        throw new Error("Unknown product type");
    }
  }
}
```

### Observer Pattern

```typescript
interface Observer {
  update(data: any): void;
}

class Subject {
  private observers: Observer[] = [];

  attach(observer: Observer): void {
    this.observers.push(observer);
  }

  notify(data: any): void {
    this.observers.forEach(observer => observer.update(data));
  }
}
```

---

- Go back to [Languages](./index.md)
- Return to [Home](../../index.md)
