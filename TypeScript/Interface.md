# Interface

## Definition / Concept
An **interface** in TypeScript is a way to define a contract for the shape of objects. Interfaces define properties and methods that an object must have. Unlike types, **interfaces are reopenable** and can be **extended** multiple times, making them ideal for **object-oriented programming**. Interfaces are **structural types**, meaning they match objects based on their shape, not their declaration.

- Define object structures and contracts
- Support **inheritance** and **extension**
- Can be **merged** or **reopened** to add new properties
- Cannot be used for primitives, unions, or tuples

## Visual Representation

```
Interface Declaration:
┌──────────────────────────────────┐
│  interface User {                │
│    id: number;                   │
│    name: string;                 │
│    email?: string;               │
│    getDetails(): string;         │
│  }                               │
└──────────────────────────────────┘

Interface Inheritance:
┌─────────────────┐
│  interface User │
│    {id, name}   │
└────────┬────────┘
         │ extends
┌────────▼──────────────┐
│ interface Admin       │
│   {permissions}      │
└──────────────────────┘

Interface Merging:
┌──────────────────┐    ┌──────────────────┐
│ interface Config │    │ interface Config │
│  {theme: string} │ +  │  {apiUrl: string}│
│                  │    │  (merged)        │
└──────────────────┘    └──────────────────┘
```

## Example

```typescript
// Basic interface
interface User {
  id: number;
  name: string;
  email?: string;
}

// Interface extending another
interface Admin extends User {
  role: 'admin';
  permissions: string[];
}

// Implementing interface in class
class ConsoleLogger implements Logger {
  log(message: string): void {
    console.log(message);
  }
}

// Using the interface
const admin: Admin = {
  id: 1,
  name: 'Bob',
  role: 'admin',
  permissions: ['read', 'write']
};
```

## Usage
- **When to use**: Defining contracts for object-oriented code, class implementations, and when you need to extend or merge types
- **Real-world example**: Defining API models, component props interfaces, service contracts, database schemas
- **Best practices**:
  - Use `interface` for object-oriented patterns
  - Use `interface` when you need to extend or merge definitions
  - Keep interfaces focused on a single responsibility
  - Use `extends` for inheritance rather than creating large monolithic interfaces
  - Implement interfaces in classes to enforce structure
  - Export interfaces from dedicated files for reusability

## FAQ / Interview Questions

**Q: What is the difference between `interface` and `type`?**
A: The main differences are:
- `interface` uses declaration syntax, `type` uses `=`
- `interface` can be reopened and merged; `type` cannot
- `interface` only works with objects; `type` supports unions, tuples, primitives
- `interface` supports `implements` in classes; `type` doesn't
- `interface` has better error messages in many cases
- Choose `interface` for OOP, `type` for functional patterns

**Q: What is declaration merging in TypeScript?**
A: Declaration merging allows you to define the same interface multiple times, and TypeScript will combine all declarations into a single interface:
```typescript
interface User {
  name: string;
}

interface User {
  email: string;
}

// Merged result: { name: string; email: string }
const user: User = { name: 'John', email: 'john@example.com' };
```

**Q: Can you show how interface inheritance works?**
A: Interfaces can extend one or more interfaces using the `extends` keyword:
```typescript
interface Animal {
  name: string;
  age: number;
}

interface Dog extends Animal {
  breed: string;
  bark(): void;
}

const dog: Dog = {
  name: 'Rex',
  age: 5,
  breed: 'Labrador',
  bark: () => console.log('Woof!')
};
```

**Q: How do you implement an interface in a class?**
A: Use the `implements` keyword to enforce that a class satisfies an interface contract:
```typescript
interface Vehicle {
  start(): void;
  stop(): void;
  speed: number;
}

class Car implements Vehicle {
  speed: number = 0;

  start(): void {
    console.log('Car started');
    this.speed = 10;
  }

  stop(): void {
    console.log('Car stopped');
    this.speed = 0;
  }
}

const car = new Car();
car.start();
```

**Q: When would you use interface over type?**
A:
- Use `interface` when defining object contracts for classes
- Use `interface` when you need declaration merging (e.g., extending third-party types)
- Use `interface` for clear OOP patterns
- Use `interface` for DOM and library definitions
- Use `type` for functional utilities, unions, and conditional types

## References
- [TypeScript Handbook - Interfaces](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [TypeScript Documentation - Declaration Merging](https://www.typescriptlang.org/docs/handbook/declaration-merging.html)
- [Interfaces vs Types - TypeScript Guide](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html#interfaces-vs-type-aliases)
- [TypeScript Deep Dive - Interfaces](https://basarat.gitbook.io/typescript/type-system/interfaces)

---
*See also: [Type](Type.md), [Classes](Classes.md), [Generics](Generics.md)*
