# Type Guards in TypeScript

## Definition / Concept

**Type guards** are conditional checks that help TypeScript **narrow down** the type of a variable within a specific code block. When you have a value with a broad or union type, type guards allow TypeScript to understand that within certain code paths, the value must be a more specific type. This enables you to write type-safe code without explicit type casting and prevents runtime errors. Common type guards include `typeof`, `instanceof`, custom type predicates, and `in` operator checks, each serving different purposes for narrowing types.

- **Type narrowing**: Refining a broad type into a more specific type
- **Type safety**: Prevent errors by ensuring type checks before access
- **Control flow analysis**: TypeScript tracks type changes within code blocks
- **Custom predicates**: User-defined functions that narrow types
- **Exhaustiveness checking**: Ensure all type possibilities are handled

## Visual Representation

```
TYPE GUARD NARROWING FLOW

Value: string | number | null
         │
         ├─────────────────────────────────────┐
         │                                     │
    ┌────▼────┐                          ┌─────▼──────┐
    │ typeof  │                          │ instanceof │
    │ check   │                          │ check      │
    └────┬────┘                          └─────┬──────┘
         │                                     │
    ┌────┴─────────┐                   ┌──────┴──────┐
    │              │                   │             │
  string        number              Error          Date
                                    object       custom class
                    │
    ┌───────────────┼───────────────┐
    │               │               │
  "string"      42                null
    │               │
  string         number

OTHER GUARDS:
├─ in operator: 'property' in obj
├─ truthy checks: if (value)
├─ custom predicates: is X
└─ discriminated unions: obj.type === 'A'

CONTROL FLOW ANALYSIS:
function process(value: string | number) {
  if (typeof value === 'string') {
    // Inside: value is string ✓
  } else {
    // Inside: value is number ✓
  }
  // Outside: value is string | number
}
```

## Example

```typescript
// ===== TYPEOF GUARD =====

function processValue(value: string | number): void {
  if (typeof value === 'string') {
    // ✅ TypeScript knows value is string here
    console.log(value.toUpperCase());
  } else {
    // ✅ TypeScript knows value is number here
    console.log(value.toFixed(2));
  }
}

// ===== INSTANCEOF GUARD =====

class Dog {
  bark() { console.log("Woof!"); }
}

class Cat {
  meow() { console.log("Meow!"); }
}

function makeSound(animal: Dog | Cat): void {
  if (animal instanceof Dog) {
    // ✅ TypeScript knows animal is Dog
    animal.bark();
  } else {
    // ✅ TypeScript knows animal is Cat
    animal.meow();
  }
}

// ===== IN OPERATOR GUARD =====

interface User {
  name: string;
  email: string;
}

interface Admin extends User {
  permissions: string[];
}

function describe(person: User | Admin): string {
  if ('permissions' in person) {
    // ✅ TypeScript knows person is Admin
    return `${person.name} (Admin with ${person.permissions.length} perms)`;
  }
  // ✅ TypeScript knows person is User
  return `${person.name} (Regular user)`;
}

// ===== CUSTOM TYPE PREDICATE =====

interface Bird {
  fly(): void;
  sing(): void;
}

interface Fish {
  swim(): void;
}

// Type predicate function
function isBird(animal: Bird | Fish): animal is Bird {
  return 'fly' in animal && 'sing' in animal;
}

function move(animal: Bird | Fish): void {
  if (isBird(animal)) {
    // ✅ TypeScript knows animal is Bird
    animal.fly();
    animal.sing();
  } else {
    // ✅ TypeScript knows animal is Fish
    animal.swim();
  }
}

// ===== TRUTHINESS GUARD =====

function printLength(str: string | null): void {
  if (str) {
    // ✅ TypeScript knows str is string (null is excluded)
    console.log(str.length);
  } else {
    // ✅ TypeScript knows str is null
    console.log("No string provided");
  }
}

// ===== EQUALITY GUARD =====

type Status = 'success' | 'error' | 'pending';

function handleStatus(status: Status): void {
  if (status === 'success') {
    // ✅ TypeScript narrows to 'success' literal
    console.log("Operation successful!");
  } else if (status === 'error') {
    // ✅ TypeScript narrows to 'error' literal
    console.log("Operation failed");
  } else {
    // ✅ TypeScript narrows to 'pending' literal
    console.log("Still processing...");
  }
}

// ===== DISCRIMINATED UNION =====

type Result =
  | { status: 'success'; data: string }
  | { status: 'error'; message: string };

function processResult(result: Result): string {
  if (result.status === 'success') {
    // ✅ TypeScript knows result has data property
    return result.data;
  } else {
    // ✅ TypeScript knows result has message property
    return result.message;
  }
}

// ===== TYPE PREDICATE WITH GENERIC =====

function isNotNull<T>(value: T | null): value is T {
  return value !== null;
}

const values: (string | null)[] = ["hello", null, "world"];
const filtered: string[] = values.filter(isNotNull);
// ✅ TypeScript knows filtered contains only strings

// ===== EXHAUSTIVENESS CHECKING =====

type Animal = 'dog' | 'cat' | 'bird';

function getSound(animal: Animal): string {
  switch (animal) {
    case 'dog':
      return "Woof";
    case 'cat':
      return "Meow";
    case 'bird':
      return "Tweet";
    default:
      // ✅ TypeScript error if Animal type changes
      // and new case isn't handled
      const _exhaustive: never = animal;
      return _exhaustive;
  }
}

// ===== OPTIONAL CHAINING WITH GUARD =====

interface Config {
  api?: {
    url?: string;
    timeout?: number;
  };
}

function getApiUrl(config: Config): string {
  // Optional chaining automatically narrows undefined
  const url = config.api?.url;

  if (url) {
    return url;  // ✅ url is string here
  }
  return 'https://api.example.com';
}

// ===== NULLISH COALESCING WITH GUARD =====

function setDefault(value: string | null | undefined): string {
  // Nullish coalescing narrows null/undefined
  return value ?? 'default value';  // Returns default if null or undefined
}
```

## Usage

- **When to use type guards**:
  - Handling union types safely
  - Working with optional properties
  - Implementing polymorphic behavior
  - Narrowing types before accessing properties/methods
  - Processing different API response formats
  - Building discriminated union handlers

- **Real-world example**:
  ```typescript
  // API response handler with type guards
  type ApiResponse =
    | { status: 200; body: { id: number; name: string } }
    | { status: 400; error: string }
    | { status: 500; error: string; details?: string };

  async function handleResponse(response: Response): Promise<void> {
    const data: ApiResponse = await response.json();

    if (data.status === 200) {
      // ✅ TypeScript knows body exists
      console.log(`User: ${data.body.name}`);
    } else if (data.status === 400) {
      // ✅ TypeScript knows error exists
      console.log(`Bad request: ${data.error}`);
    } else {
      // ✅ TypeScript knows status is 500, details is optional
      console.log(`Server error: ${data.error}`);
      if (data.details) {
        console.log(`Details: ${data.details}`);
      }
    }
  }

  // Custom type guard for validation
  interface User {
    name: string;
    email: string;
  }

  function isValidUser(data: unknown): data is User {
    return (
      typeof data === 'object' &&
      data !== null &&
      'name' in data &&
      'email' in data &&
      typeof data.name === 'string' &&
      typeof data.email === 'string'
    );
  }

  async function createUser(data: unknown): Promise<void> {
    if (!isValidUser(data)) {
      throw new Error('Invalid user data');
    }
    // ✅ data is User here
    console.log(`Creating user: ${data.name}`);
  }
  ```

- **Best practices**:
  - Use **type predicates** (`is`) for reusable type narrowing logic
  - Combine **truthiness checks** with optional chaining (`?.`) for null safety
  - Use **discriminated unions** for complex objects with multiple variants
  - Implement **exhaustiveness checks** with `never` to catch missing cases
  - Create **guard utilities** for common validation patterns
  - Avoid negating type guards when possible (harder to narrow)
  - Document complex guards with JSDoc for clarity
  - Test type guards thoroughly to ensure correctness

## FAQ / Interview Questions

**Q: What is a type guard and how does it differ from type casting?**
A: A type guard is a runtime check that narrows a type for TypeScript's type checker, while type casting (using `as`) forces TypeScript to treat a value as a different type without checking. Type guards are safer because TypeScript verifies the check—if the runtime check is true, the type is actually narrowed. Casting bypasses the checker, which can lead to runtime errors.

**Q: What's the difference between `typeof` and `instanceof` guards?**
A: `typeof` checks primitive types and returns a string ('string', 'number', 'boolean', 'object', 'undefined', 'function'). It works with primitives like strings, numbers, booleans, and undefined. `instanceof` checks if an object is an instance of a class or constructor function. Use `typeof` for primitives, `instanceof` for class instances and built-in objects like `Date`, `Error`, `RegExp`.

**Q: What is a type predicate and why use it?**
A: A type predicate is a custom function with return type `value is Type` that narrows the type. Example: `function isString(x): x is string { return typeof x === 'string' }`. Use predicates when you need reusable narrowing logic, especially for complex validation. They enable you to extract type narrowing into utility functions, making code more maintainable and testable.

**Q: How do discriminated unions work as type guards?**
A: A discriminated union has a common **literal property** that identifies which variant it is. When you check that property, TypeScript automatically narrows the type:
```typescript
type Result = { kind: 'success'; value: string } | { kind: 'error'; message: string };
if (result.kind === 'success') {
  result.value;  // ✅ TypeScript knows value exists
}
```
This is cleaner than `instanceof` for interfaces and more flexible than separate checks.

**Q: What's the purpose of the `never` type in exhaustiveness checking?**
A: The `never` type represents an impossible value—a variable can never have this type. In a switch statement, if you assign the remaining case to `never`, TypeScript ensures all cases are handled. If you add a new case to a union but forget to handle it in the switch, TypeScript errors because the unhandled case can't be assigned to `never`.

**Q: How do optional chaining and nullish coalescing help with type safety?**
A: Optional chaining (`?.`) safely accesses nested properties and returns `undefined` if the value is null/undefined. Nullish coalescing (`??`) returns an alternative value only for `null`/`undefined`, not other falsy values. Together, they provide safe navigation without manual null checks: `config.api?.url ?? 'default'`. They work with type narrowing for cleaner, safer code.

## References

- [TypeScript Handbook - Type Guards](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TypeScript Handbook - Discriminated Unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
- [TypeScript Handbook - Using Type Predicates](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#using-type-predicates)
- [TypeScript Deep Dive - Type Guards](https://basarat.gitbook.io/typescript/type-system/type-guards-and-differentiating-types)

---

*See also: [Types](./Types.md), [TypeSystem](./TypeSystem.md), [Generics](./Generics.md), [Interface](./Interface.md)*
