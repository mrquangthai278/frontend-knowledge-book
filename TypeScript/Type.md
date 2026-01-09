# Type

## Definition / Concept
A **type** in TypeScript is a way to define the shape of data using the `type` keyword. Types are used to create type aliases for primitives, unions, tuples, and complex objects. Unlike interfaces, types are more flexible and can represent any value, including primitive types. Types are **immutable** and support advanced features like **conditional types** and **mapped types**.

- Define custom data shapes using type aliases
- Support union types (`|`) and intersection types (`&`)
- Cannot be reopened or extended after declaration
- Ideal for functional programming patterns

## Visual Representation

```
Type Declaration:
┌─────────────────────────────────┐
│  type UserProfile = {           │
│    name: string;                │
│    age: number;                 │
│    isActive?: boolean;          │
│  }                              │
└─────────────────────────────────┘

Union Type:
┌──────────────────────────┐
│ type Status =            │
│   | 'pending'            │
│   | 'success'            │
│   | 'error'              │
└──────────────────────────┘

Intersection Type:
┌──────────────────────────┐
│ type Admin = User & {    │
│   permissions: string[]; │
│ }                        │
└──────────────────────────┘
```

## Example

```typescript
// Basic type alias
type User = {
  id: number;
  name: string;
  email: string;
};

// Union type - value can be one of several types
type Status = 'pending' | 'completed' | 'failed';

// Intersection type - combines multiple types
type Admin = User & {
  role: 'admin';
  permissions: string[];
};

// Function type
type Callback = (data: string) => void;

// Conditional type
type IsString<T> = T extends string ? true : false;
type Result = IsString<'hello'>; // true

// Mapped type - transforms properties of an existing type
type Readonly<T> = {
  readonly [K in keyof T]: T[K];
};

// Using types in practice
const user: User = {
  id: 1,
  name: 'John',
  email: 'john@example.com'
};

const updateStatus = (status: Status) => {
  console.log(`Status is ${status}`);
};

updateStatus('completed'); // Valid
// updateStatus('unknown'); // Error: not a valid Status
```

## Usage
- **When to use**: Creating type aliases for complex data structures, union types, function signatures, and advanced type utilities
- **Real-world example**: Defining API response shapes, union types for state machines, conditional types for generic utilities
- **Best practices**:
  - Use `type` for functional/data-focused structures
  - Use `type` for unions and tuples
  - Keep types focused and single-responsibility
  - Export types for reuse across modules
  - Use meaningful names that describe the data shape

## FAQ / Interview Questions

**Q: What's the difference between `type` and `interface`?**
A: The main differences are:
- `type` uses `=`, `interface` uses declaration syntax
- `type` cannot be reopened; `interface` can be extended/merged
- `type` supports unions, tuples, and primitives; `interface` only objects
- `type` supports mapped and conditional types
- `interface` is better for OOP patterns; `type` for functional patterns

**Q: Can you give an example of a union type?**
A: Union types allow a value to be one of several types:
```typescript
type Response =
  | { status: 'success'; data: unknown }
  | { status: 'error'; error: string };

const handleResponse = (res: Response) => {
  if (res.status === 'success') {
    console.log(res.data);
  } else {
    console.log(res.error);
  }
};
```

**Q: What are conditional types and when would you use them?**
A: Conditional types allow you to select a type based on a condition. They use the syntax `T extends U ? X : Y`. Example:
```typescript
type IsString<T> = T extends string ? 'yes' : 'no';
type Result = IsString<'hello'>; // 'yes'
```
Use cases: Creating generic utilities, type guards, conditional API responses.

**Q: How do mapped types work?**
A: Mapped types iterate over properties of an existing type to create a new type:
```typescript
type Nullable<T> = {
  [K in keyof T]: T[K] | null;
};

type User = { name: string; age: number };
type NullableUser = Nullable<User>;
// Result: { name: string | null; age: number | null }
```

**Q: What are the advantages of using types over inline object definitions?**
A:
- Reusability across multiple functions and components
- Easier maintenance and refactoring
- Better IDE autocomplete and type checking
- Self-documenting code
- Ability to compose types together

## References
- [TypeScript Handbook - Type Aliases](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [TypeScript Documentation - Creating Types from Types](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [MDN - TypeScript Types](https://developer.mozilla.org/en-US/docs/Glossary/TypeScript)
- [Advanced TypeScript Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)

---
*See also: [Interface](Interface.md), [Union Types](UnionTypes.md), [Generics](Generics.md)*
