# Generics in TypeScript

## Definition / Concept

**Generics** are a way to create reusable components that work with different types while maintaining **type safety**. A generic is a placeholder for a type that is specified when the generic is used, allowing you to write flexible code without sacrificing type checking. Instead of writing separate functions or classes for each type, generics let you use **type variables** (usually `T`, `U`, `K`) to represent unknown types that will be filled in later. This eliminates code duplication and provides flexibility with full type safety.

- **Type variables**: Placeholders like `T`, `U`, `K` that represent unknown types
- **Generic constraints**: Restrictions on what types a generic can accept
- **Type inference**: TypeScript automatically infers generic types from arguments
- **Reusability**: Write once, use with many different types safely

## Visual Representation

```
GENERICS FLOW ARCHITECTURE

┌──────────────────────────────────────────────────────┐
│         Defining Generic Component                    │
│         function box<T>(value: T): T { ... }         │
└──────────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌─────────┐     ┌─────────┐    ┌──────────┐
   │ IMPLICIT│     │ EXPLICIT │    │CONSTRAINED
   │INFERENCE│     │SPECIFICATION│   │GENERICS
   └─────────┘     └─────────┘    └──────────┘
        │               │              │
   box(5)          box<number>(5) box<T extends
   T inferred      T = number     string>(value)
   as number

┌──────────────────────────────────────────────────────┐
│      Generic Constraint Hierarchy                     │
└──────────────────────────────────────────────────────┘

    T extends any
         │
    ┌────┼────┐
    │         │
T extends   T extends
  {length}   | keyof T

    Complex types can be:
    - Single type: T extends string
    - Union: T extends string | number
    - Interface: T extends { id: number }
    - Keyof: T extends keyof U
```

## Example

```typescript
// ===== BASIC GENERICS =====

// Generic function with type inference
function identity<T>(value: T): T {
  return value;
}

// Explicit type specification
const numResult: number = identity<number>(42);
const strResult: string = identity<string>("hello");

// Implicit type inference
const inferred = identity(true);  // T inferred as boolean

// ===== GENERIC WITH MULTIPLE TYPE VARIABLES =====

function pair<T, U>(first: T, second: U): [T, U] {
  return [first, second];
}

const result = pair<string, number>("age", 30);
const inferred2 = pair(true, "yes");  // T = boolean, U = string

// ===== GENERIC CONSTRAINTS =====

// Constraint to object with 'length' property
function getLength<T extends { length: number }>(value: T): number {
  return value.length;
}

getLength("hello");  // ✅ strings have length
getLength([1, 2, 3]);  // ✅ arrays have length
getLength({ length: 5, name: "obj" });  // ✅ works
// getLength(42);  // ❌ numbers don't have length

// ===== KEYOF CONSTRAINT =====

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const user = { name: "Alice", age: 30 };
const name = getProperty(user, "name");  // ✅ name is string
// getProperty(user, "email");  // ❌ email not in user

// ===== GENERIC CLASSES =====

class Stack<T> {
  private items: T[] = [];

  push(value: T): void {
    this.items.push(value);
  }

  pop(): T | undefined {
    return this.items.pop();
  }

  isEmpty(): boolean {
    return this.items.length === 0;
  }
}

const numberStack = new Stack<number>();
numberStack.push(1);
numberStack.push(2);
const popped: number | undefined = numberStack.pop();

// ===== GENERIC INTERFACES =====

interface Repository<T> {
  findById(id: number): T | null;
  save(item: T): void;
  getAll(): T[];
}

interface Product {
  id: number;
  name: string;
  price: number;
}

class ProductRepository implements Repository<Product> {
  private products: Product[] = [];

  findById(id: number): Product | null {
    return this.products.find(p => p.id === id) || null;
  }

  save(item: Product): void {
    this.products.push(item);
  }

  getAll(): Product[] {
    return this.products;
  }
}

// ===== DEFAULT GENERIC TYPES =====

interface ApiResponse<T = any> {
  success: boolean;
  data: T;
  timestamp: number;
}

const defaultResponse: ApiResponse = {
  success: true,
  data: "anything",
  timestamp: Date.now()
};

const typedResponse: ApiResponse<{ id: number }> = {
  success: true,
  data: { id: 1 },
  timestamp: Date.now()
};

// ===== CONDITIONAL GENERICS =====

type IsString<T> = T extends string ? true : false;

type A = IsString<"hello">;  // true
type B = IsString<number>;   // false

// Practical use case
function processValue<T>(value: T): T extends string ? string : number {
  return typeof value === 'string'
    ? (value as any)
    : (0 as any);
}

// ===== GENERIC MAPPED TYPES =====

// Make all properties optional
type Partial<T> = {
  [K in keyof T]?: T[K];
};

interface User {
  id: number;
  name: string;
  email: string;
}

const partialUser: Partial<User> = {
  name: "Bob"  // ✅ other properties optional
};

// ===== GENERIC CONSTRAINTS WITH EXTENDS =====

function merge<T extends object, U extends object>(
  obj1: T,
  obj2: U
): T & U {
  return { ...obj1, ...obj2 };
}

const merged = merge(
  { a: 1, b: 2 },
  { c: 3, d: 4 }
);
// merged has type: { a: number; b: number; c: number; d: number }
```

## Usage

- **When to use generics**:
  - Creating reusable utility functions and classes
  - Working with collections (arrays, maps, sets)
  - Building flexible APIs that work with different data types
  - Creating type-safe wrappers around data
  - Implementing design patterns (Builder, Factory, Strategy)

- **Real-world example**:
  ```typescript
  // Generic API client for different endpoints
  interface ApiClient<T> {
    fetch(id: number): Promise<T>;
    fetchAll(): Promise<T[]>;
    create(data: Partial<T>): Promise<T>;
    update(id: number, data: Partial<T>): Promise<T>;
    delete(id: number): Promise<void>;
  }

  class HttpClient<T> implements ApiClient<T> {
    constructor(private baseUrl: string) {}

    async fetch(id: number): Promise<T> {
      const response = await fetch(`${this.baseUrl}/${id}`);
      return response.json();
    }

    async fetchAll(): Promise<T[]> {
      const response = await fetch(this.baseUrl);
      return response.json();
    }

    async create(data: Partial<T>): Promise<T> {
      const response = await fetch(this.baseUrl, {
        method: 'POST',
        body: JSON.stringify(data)
      });
      return response.json();
    }

    async update(id: number, data: Partial<T>): Promise<T> {
      const response = await fetch(`${this.baseUrl}/${id}`, {
        method: 'PATCH',
        body: JSON.stringify(data)
      });
      return response.json();
    }

    async delete(id: number): Promise<void> {
      await fetch(`${this.baseUrl}/${id}`, { method: 'DELETE' });
    }
  }

  // Use with any type
  const userClient = new HttpClient<User>('/api/users');
  const users = await userClient.fetchAll();
  ```

- **Best practices**:
  - Keep generic constraints as broad as possible while still being useful
  - Use meaningful type variable names (`T` for Type, `K` for Key, `V` for Value)
  - Leverage type inference to avoid explicit type specifications when possible
  - Use `extends` to constrain generics and provide better error messages
  - Combine generics with utility types (`Partial`, `Pick`, `Omit`, `Record`)
  - Document generic parameters with JSDoc for clarity
  - Avoid overly complex nested generics—break them into simpler pieces

## FAQ / Interview Questions

**Q: What are generics and why are they useful?**
A: Generics are placeholders for types that are specified when a generic component is used. They allow you to write reusable, type-safe code without code duplication. For example, `Array<T>` works with any type T—`Array<string>`, `Array<number>`, etc. This prevents the need to write separate functions for each type while maintaining type safety.

**Q: What's the difference between implicit and explicit generic type specification?**
A: Implicit type specification relies on **type inference**, where TypeScript automatically determines the generic type from arguments: `identity(42)` infers `T` as `number`. Explicit specification passes the type directly: `identity<number>(42)`. Implicit is usually preferred because it's cleaner, but explicit specification is useful when inference fails or when you want to be explicit about intent.

**Q: What does `extends` mean in the context of generics?**
A: `extends` in generics creates **constraints** on what types the generic can accept. For example, `<T extends string>` means T can only be a string or subtype of string. `<T extends { length: number }>` means T must be an object with a `length` property. Constraints ensure the generic is only used with compatible types and enable better error messages.

**Q: How do you use generics with arrays and other built-in types?**
A: TypeScript's built-in types like `Array`, `Promise`, `Map`, etc. are already generics. You specify the type in angle brackets: `Array<number>`, `Promise<string>`, `Map<string, User>`. You can also create generic versions of these yourself or use the `<T>` syntax with your own classes and functions.

**Q: What are utility types and how do they relate to generics?**
A: Utility types are built-in generic types that transform existing types. Examples include:
- `Partial<T>`: Makes all properties optional
- `Pick<T, K>`: Selects specific properties
- `Omit<T, K>`: Excludes specific properties
- `Record<K, T>`: Creates an object with keys of type K and values of type T

These are implemented using generics and mapped types, and they help create variations of types without duplication.

**Q: Can you provide a practical example of generic constraints?**
A: A common example is filtering a generic array to return items matching a property:
```typescript
function filterByProperty<T, K extends keyof T>(
  items: T[],
  key: K,
  value: T[K]
): T[] {
  return items.filter(item => item[key] === value);
}

const users: User[] = [{ id: 1, name: "Alice" }, { id: 2, name: "Bob" }];
const filtered = filterByProperty(users, "name", "Alice");  // ✅ Works, "name" exists
// filterByProperty(users, "email", "");  // ❌ Error, "email" doesn't exist on User
```
This ensures type safety while remaining flexible.

## References

- [TypeScript Handbook - Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
- [TypeScript Handbook - Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
- [TypeScript Handbook - Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [TypeScript Deep Dive - Generics](https://basarat.gitbook.io/typescript/type-system/generics)

---

*See also: [Types](./Types.md), [Type](./Type.md), [Interface](./Interface.md), [TypeSystem](./TypeSystem.md)*
