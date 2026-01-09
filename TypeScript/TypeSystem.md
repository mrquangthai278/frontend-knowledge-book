# TypeScript Type System (Structural and Nominal Typing)

## Definition / Concept

**TypeScript's type system** is a set of rules that determines how types are compared and checked for compatibility. TypeScript uses **structural typing** (also called duck typing), where two types are considered compatible if their structure matches, regardless of their declared names. This contrasts with **nominal typing**, where types are compatible only if explicitly declared to be the same type. Understanding this distinction is crucial because it affects how types interact, interface compatibility, and type safety in your code.

- **Structural typing**: Types are compatible based on shape/structure, not declaration
- **Nominal typing**: Types are compatible only if explicitly named as the same type
- **Duck typing principle**: "If it walks like a duck and quacks like a duck, it's a duck"
- **Type compatibility**: Determines when one type can be assigned to another

## Visual Representation

```
TypeScript Type System Architecture

┌─────────────────────────────────────────────────────┐
│          Type Compatibility Checking                 │
└─────────────────────────────────────────────────────┘
                        │
        ┌───────────────┴────────────────┐
        │                                │
   ┌─────────────┐             ┌──────────────────┐
   │ STRUCTURAL  │             │    NOMINAL       │
   │   TYPING    │             │    TYPING        │
   └─────────────┘             └──────────────────┘
   (TypeScript)                (Other languages)
        │                              │
        │ Shape-based                  │
        │ compatibility                │ Explicit declaration
        │                              │ required
        ├─ Duck typing                 │
        ├─ Assignable by               │
        │  matching properties         │
        └─ Runtime structure           │
           matters, not name           │

Example:
interface Dog { bark(): void }
interface Cat { bark(): void }

// Structurally compatible ✅
const animal: Dog = {} as Cat;

// Nominally incompatible ❌
// (if TS had nominal typing)
```

## Example

```typescript
// ===== STRUCTURAL TYPING (TypeScript Default) =====

interface User {
  name: string;
  age: number;
}

interface Person {
  name: string;
  age: number;
}

// ✅ Structurally compatible - same shape
const user: User = { name: "Alice", age: 30 };
const person: Person = user;  // No error!

// ✅ Assignable even without explicit type declaration
const obj = { name: "Bob", age: 25 };
const assignedUser: User = obj;  // Works - structure matches

// ===== STRUCTURAL TYPING WITH EXTRA PROPERTIES =====

interface Employee {
  name: string;
  salary: number;
}

const developer = {
  name: "Charlie",
  salary: 100000,
  language: "TypeScript"  // extra property
};

// ✅ Still assignable - Employee properties are present
const emp: Employee = developer;

// ===== DUCK TYPING IN PRACTICE =====

interface Logger {
  log(message: string): void;
}

class ConsoleLogger {
  log(message: string) {
    console.log(message);
  }
}

class FileLogger {
  log(message: string) {
    // write to file
  }
}

// ✅ Both work without implementing Logger interface
const logger1: Logger = new ConsoleLogger();
const logger2: Logger = new FileLogger();

// ===== NOMINAL TYPING WORKAROUND (Type Brands) =====

// Create unique types using intersection with nominal marker
type UserId = string & { readonly __brand: "UserId" };
type ProductId = string & { readonly __brand: "ProductId" };

// Helper to create branded types safely
function createUserId(id: string): UserId {
  return id as UserId;
}

function createProductId(id: string): ProductId {
  return id as ProductId;
}

const userId: UserId = createUserId("user-123");
const productId: ProductId = createProductId("prod-456");

// ❌ Type error - incompatible branded types
// const result: UserId = productId;

// ===== STRUCTURAL COMPATIBILITY RULES =====

interface Source {
  x: number;
  y: number;
}

interface Target {
  x: number;
  y: number;
  z?: number;  // optional property
}

// ✅ Source assignable to Target (Target has optional z)
const source: Source = { x: 1, y: 2 };
const target: Target = source;

// ❌ But not vice versa - Target might not have x, y
// const rev: Source = { x: 1, y: 2, z: 3 };  // error if z required
```

## Usage

- **When to use structural typing**:
  - Building flexible APIs and libraries
  - Creating duck-typed interfaces that work with compatible types
  - Reducing boilerplate and explicit type declarations
  - Working with third-party types that share the same shape

- **Real-world example**:
  ```typescript
  // Multiple data sources with same shape
  interface ApiResponse {
    data: any;
    status: number;
    error?: string;
  }

  // From HTTP library
  const httpResponse = { data: {...}, status: 200 };

  // From database query
  const dbResponse = { data: {...}, status: 200 };

  // Both assignable to ApiResponse due to structural typing
  function handleResponse(res: ApiResponse) {
    if (res.error) console.error(res.error);
  }

  handleResponse(httpResponse);  // ✅ Works
  handleResponse(dbResponse);    // ✅ Works
  ```

- **Best practices**:
  - Use **type brands** when you need nominal typing (IDs, tokens, etc.)
  - Be aware that structural typing allows accidental compatibility
  - Document expected type shapes clearly in interfaces
  - Use discriminated unions to distinguish similar types
  - Leverage `Pick<T, K>` and other utility types for precise compatibility
  - Test that your types catch intended errors despite structural compatibility

## FAQ / Interview Questions

**Q: What is the main difference between structural and nominal typing?**
A: Structural typing checks type compatibility based on shape (properties and methods), while nominal typing requires explicit type declarations to match. TypeScript uses structural typing—two interfaces are compatible if they have the same structure, regardless of their names. Nominal typing (like in Java or C#) requires explicit inheritance or type declaration.

**Q: Why does TypeScript use structural typing instead of nominal typing?**
A: Structural typing is more flexible and reduces boilerplate code. It allows you to work with types from different sources that happen to have compatible shapes without explicit type relationships. This is particularly useful for working with third-party libraries and creating polymorphic code. However, it requires careful design to avoid accidental type compatibility.

**Q: Can you create nominal typing in TypeScript?**
A: Yes, using **type brands** (also called opaque types). Create a unique marker type using intersection with a phantom property:
```typescript
type UserId = string & { readonly __brand: "UserId" };
function createUserId(id: string): UserId { return id as UserId; }
```
This ensures only properly created `UserId` values are assignable to the type, adding nominal-like semantics to structural typing.

**Q: What happens when a type has extra properties compared to the target interface?**
A: In structural typing, a type with extra properties is still assignable to a target interface as long as it has all required properties. For example, `{ x: 1, y: 2, z: 3 }` is assignable to an interface expecting only `{ x: number, y: number }` because it has the required properties.

**Q: How do you distinguish between similar types with the same structure?**
A: Use **discriminated unions** (tagged unions) with a literal property:
```typescript
type Success = { status: 'success'; data: any };
type Error = { status: 'error'; message: string };
type Result = Success | Error;

function handle(result: Result) {
  if (result.status === 'success') {
    // TypeScript knows this is Success
  }
}
```

## References

- [TypeScript Handbook - Structural Subtyping](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)
- [TypeScript Handbook - Type Compatibility](https://www.typescriptlang.org/docs/handbook/2/types-from-types.html)
- [Nominal vs Structural Typing - TypeScript Deep Dive](https://basarat.gitbook.io/typescript/type-system/type-compatibility)
- [Branded Types / Phantom Types Pattern](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)

---

*See also: [Types](./Types.md), [Interface](./Interface.md), [Type](./Type.md), [StrictMode](./StrictMode.md)*
