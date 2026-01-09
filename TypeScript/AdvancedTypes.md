# Advanced Types in TypeScript

## Definition / Concept

**Advanced Types** in TypeScript are sophisticated type constructs that go beyond basic types, enabling you to express complex type relationships and transformations. These include **union types**, **intersection types**, **conditional types**, **mapped types**, and **utility types**. Advanced types allow you to create flexible, reusable type definitions that adapt to different scenarios, enforce constraints, and manipulate existing types to generate new ones. Mastering these enables you to write more expressive, maintainable, and type-safe code.

- **Union types**: Multiple possible types combined with `|`
- **Intersection types**: Combine multiple types with `&`
- **Conditional types**: Types that depend on other types
- **Mapped types**: Transform existing types into new ones
- **Utility types**: Built-in generic types for common transformations
- **Template literal types**: Literal types based on string templates

## Visual Representation

```
ADVANCED TYPES HIERARCHY

┌─────────────────────────────────────────────────┐
│         Advanced Type Constructs                 │
└─────────────────────────────────────────────────┘
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   ┌─────────────┐ ┌────────────┐ ┌──────────────┐
   │   UNION     │ │ INTERSECTION
│ │  CONDITIONAL │
   │ A | B      │ │ A & B      │ │  T extends U │
   └─────────────┘ └────────────┘ └──────────────┘
        │               │               │
   Multiple types   Combined props   Type-dependent
                                       logic

MAPPED TYPES & UTILITIES
┌─────────────────────────────────┐
│ Transform: T → T'               │
├─────────────────────────────────┤
│ Partial<T>   → all optional     │
│ Required<T>  → all required     │
│ Readonly<T>  → all readonly     │
│ Pick<T, K>   → select props     │
│ Omit<T, K>   → exclude props    │
│ Record<K, V> → key-value pairs  │
│ Exclude<T, U>→ exclude types    │
│ Extract<T, U>→ extract types    │
└─────────────────────────────────┘

CONDITIONAL TYPE FLOW
┌────────────────────────────┐
│ T extends U ? X : Y        │
└────────────────────────────┘
         │
    ┌────┴────┐
    │         │
  true      false
    │         │
    X         Y
```

## Example

```typescript
// ===== UNION TYPES =====

type Status = 'pending' | 'success' | 'error';
type Primitive = string | number | boolean;

function handleStatus(status: Status): void {
  // status can only be one of these values
}

// Union with null/undefined
type Optional<T> = T | null | undefined;
const value: Optional<string> = null;  // ✅ valid

// ===== INTERSECTION TYPES =====

interface Person {
  name: string;
  age: number;
}

interface Employee {
  employeeId: string;
  department: string;
}

type Staff = Person & Employee;  // ✅ has all properties

const staff: Staff = {
  name: "Alice",
  age: 30,
  employeeId: "EMP001",
  department: "Engineering"
};

// Merging function signatures
type Callable = { (x: number): string } & { (x: string): number };

// ===== CONDITIONAL TYPES =====

type IsString<T> = T extends string ? true : false;
type A = IsString<"hello">;   // true
type B = IsString<number>;    // false

// Extracting return type
type ReturnType<T> = T extends (...args: any[]) => infer R ? R : never;
type NumReturn = ReturnType<() => number>;  // number

// Filtering tuple types
type Flatten<T> = T extends Array<infer U> ? U : T;
type Str = Flatten<string[]>;  // string
type Num = Flatten<number>;    // number

// ===== MAPPED TYPES =====

// Make all properties readonly
type ReadonlyPerson = {
  readonly [K in keyof Person]: Person[K];
};

// Make all properties optional
type PartialPerson = {
  [K in keyof Person]?: Person[K];
};

// Create getters for all properties
type Getters<T> = {
  [K in keyof T as `get${Capitalize<string & K>}`]: () => T[K];
};

type PersonGetters = Getters<Person>;
// {
//   getName: () => string;
//   getAge: () => number;
// }

// ===== TEMPLATE LITERAL TYPES =====

type EventName = `on${Capitalize<'click' | 'change' | 'submit'>}`;
// 'onClick' | 'onChange' | 'onSubmit'

type CSSValue = `${number}px` | `${number}%` | `${number}em`;
type ValidCSS = CSSValue;
const width: ValidCSS = "100px";  // ✅
// const height: ValidCSS = "100";  // ❌ Error

// ===== UTILITY TYPES =====

interface Todo {
  id: number;
  title: string;
  completed: boolean;
  dueDate?: string;
}

// Partial<T> - all properties optional
type PartialTodo = Partial<Todo>;
const draft: PartialTodo = { title: "Write docs" };  // ✅ partial

// Required<T> - all properties required
type RequiredTodo = Required<Todo>;
const complete: RequiredTodo = {
  id: 1,
  title: "Task",
  completed: true,
  dueDate: "2026-01-15"  // ✅ required
};

// Pick<T, K> - select specific properties
type TodoPreview = Pick<Todo, 'title' | 'completed'>;
const preview: TodoPreview = {
  title: "My task",
  completed: false
};

// Omit<T, K> - exclude specific properties
type TodoWithoutId = Omit<Todo, 'id'>;
const todoData: TodoWithoutId = {
  title: "Task",
  completed: false,
  dueDate: "2026-01-15"
};

// Record<K, V> - create object with specific keys and values
type TodoList = Record<'active' | 'completed', Todo[]>;
const list: TodoList = {
  active: [{ id: 1, title: "Work", completed: false }],
  completed: [{ id: 2, title: "Rest", completed: true }]
};

// Exclude<T, U> - exclude types from union
type NonNullable<T> = Exclude<T, null | undefined>;
type SafeValue = NonNullable<string | null>;  // string

// Extract<T, U> - extract types from union
type Methods = Extract<'get' | 'post' | 'put', 'get' | 'post'>;
// 'get' | 'post'

// ReturnType<T> - get function return type
type GetReturn = ReturnType<() => string[]>;  // string[]

// ===== DISTRIBUTIVE CONDITIONAL TYPES =====

type ToArray<T> = T extends any ? T[] : never;

type NumOrStr = ToArray<number | string>;
// number[] | string[] (applied to each union member)

// ===== RECURSIVE TYPES =====

type JSON =
  | string
  | number
  | boolean
  | null
  | JSON[]
  | { [key: string]: JSON };

const validJson: JSON = {
  name: "Alice",
  age: 30,
  hobbies: ["reading", "coding"],
  metadata: {
    created: "2026-01-09",
    active: true
  }
};

// ===== READONLY TYPES =====

type ReadonlyArray<T> = {
  readonly [K in keyof T[]]: T;
};

const readonlyList: readonly string[] = ["a", "b"];
// readonlyList[0] = "c";  // ❌ Error - cannot mutate

interface ReadonlyConfig {
  readonly apiUrl: string;
  readonly timeout: number;
}

// ===== KEYOF WITH GENERICS =====

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}

const person = { name: "Bob", age: 25 };
const name = getProperty(person, "name");  // ✅ string
// getProperty(person, "email");  // ❌ Error - not in person

// ===== COMPLEX UTILITY COMBINATION =====

type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

interface Config {
  api: {
    url: string;
    timeout: number;
    retry: {
      count: number;
      delay: number;
    };
  };
}

type PartialConfig = DeepPartial<Config>;
const config: PartialConfig = {
  api: {
    retry: {
      count: 3
      // delay is optional too
    }
  }
};
```

## Usage

- **When to use advanced types**:
  - Building flexible, reusable APIs and libraries
  - Creating type-safe wrappers around data transformations
  - Implementing design patterns with strong type constraints
  - Working with complex domain models
  - Extracting and manipulating type information
  - Building type-safe state management systems

- **Real-world example**:
  ```typescript
  // Form validation with advanced types
  type FormField = 'email' | 'password' | 'username';

  type FormErrors = Partial<Record<FormField, string>>;

  type FormState = {
    values: Record<FormField, string>;
    errors: FormErrors;
    touched: Partial<Record<FormField, boolean>>;
    isSubmitting: boolean;
  };

  // API response handler with conditional types
  type ApiResponse<T extends 'user' | 'post'> =
    T extends 'user'
      ? { id: number; name: string; email: string }
      : T extends 'post'
      ? { id: number; title: string; content: string }
      : never;

  async function fetchData<T extends 'user' | 'post'>(
    type: T
  ): Promise<ApiResponse<T>> {
    const response = await fetch(`/api/${type}`);
    return response.json();
  }

  // Type-safe reducer with mapped types
  type Actions = {
    SET_NAME: string;
    SET_AGE: number;
    SET_EMAIL: string;
  };

  type ReducerAction = {
    [K in keyof Actions]: { type: K; payload: Actions[K] };
  }[keyof Actions];

  function reducer(state: State, action: ReducerAction): State {
    switch (action.type) {
      case 'SET_NAME':
        return { ...state, name: action.payload };
      case 'SET_AGE':
        return { ...state, age: action.payload };
      case 'SET_EMAIL':
        return { ...state, email: action.payload };
    }
  }
  ```

- **Best practices**:
  - Use **union types** for representing multiple possible states
  - Use **intersection types** to combine related interfaces
  - Use **conditional types** for complex type logic, but keep them readable
  - Leverage **mapped types** to reduce duplication when transforming types
  - Use **utility types** first before writing custom mapped types
  - Document complex types with JSDoc for clarity
  - Break complex types into smaller, reusable pieces
  - Test utility types thoroughly with different input types
  - Use **template literal types** for type-safe string patterns
  - Consider performance—deeply recursive types can slow compilation

## FAQ / Interview Questions

**Q: What's the difference between union and intersection types?**
A: Union types (`A | B`) represent a value that can be either A or B, but not both. Intersection types (`A & B`) combine both types, so a value must satisfy both. Example: `string | number` is either string or number; `{ a: 1 } & { b: 2 }` has both `a` and `b` properties.

**Q: What are conditional types and when would you use them?**
A: Conditional types are types that depend on other types, using the syntax `T extends U ? X : Y`. They're useful for:
- Extracting types based on conditions (e.g., `ReturnType<T>`)
- Filtering types from unions (e.g., `Exclude<T, null>`)
- Creating flexible generic types that adapt to input
- Implementing type-level logic similar to if-statements

**Q: What's the difference between `Partial`, `Required`, and `Pick`?**
A: `Partial<T>` makes all properties optional; `Required<T>` makes all properties mandatory; `Pick<T, K>` selects only specified properties. Example:
```typescript
type Full = Required<Todo>;  // all props required
type Draft = Partial<Todo>;  // all props optional
type Meta = Pick<Todo, 'id' | 'title'>;  // only id and title
```

**Q: What are mapped types and how do they work?**
A: Mapped types transform properties of an existing type using the syntax `{ [K in keyof T]: /* transformation */ }`. They iterate over each property key and transform it. Example: `type Readonly<T> = { readonly [K in keyof T]: T[K] }` creates a new type with all properties readonly. They're powerful for reducing type duplication.

**Q: What are template literal types useful for?**
A: Template literal types allow you to create literal types based on string patterns. They're useful for:
- Event handler names: `onClick`, `onChange`, `onSubmit`
- CSS values: `100px`, `50%`, `2em`
- URL patterns: `/api/users/123`
- Type-safe string unions: `get${Method}` generates method names
They ensure values match specific string patterns at compile time.

**Q: How do you create a recursive type?**
A: Define a type that references itself. Common examples:
```typescript
type JSON = string | number | boolean | null | JSON[] | { [key: string]: JSON };
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};
```
Recursive types must have a base case and should avoid infinite recursion. They're useful for deeply nested structures.

## References

- [TypeScript Handbook - Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html)
- [TypeScript Handbook - Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
- [TypeScript Handbook - Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [TypeScript Handbook - Template Literal Types](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-4.html#template-literal-types)
- [TypeScript Deep Dive - Advanced Types](https://basarat.gitbook.io/typescript/type-system/advanced-types)

---

*See also: [Types](./Types.md), [Generics](./Generics.md), [TypeSystem](./TypeSystem.md), [Guards](./Guards.md), [Interface](./Interface.md)*
