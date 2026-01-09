# Types in TypeScript

## Definition / Concept

**Types** in TypeScript are a way to define what kind of data variables, parameters, and return values can hold. TypeScript adds **static typing** to JavaScript, meaning types are checked at **compile-time** before the code runs. This prevents type-related errors and improves code quality. TypeScript includes all JavaScript's primitive types plus additional types like `any`, `unknown`, `never`, `void`, tuples, unions, intersections, and custom types defined through interfaces and type aliases.

- **Static typing** is checked before code execution
- **Type inference** automatically determines types when not explicitly stated
- **Union types** allow multiple possible types
- **Type guards** narrow types in conditional branches
- **Custom types** can be defined via interfaces, classes, and type aliases

## Visual Representation

```
TYPESCRIPT TYPE SYSTEM
═══════════════════════════════════════════════════════════

┌──────────────────────────────────────────────────────────┐
│             TypeScript Type Hierarchy                     │
└──────────────────────────────────────────────────────────┘
                          │
        ┌─────────────────┼─────────────────┐
        │                 │                 │
    ┌────────────┐  ┌────────────┐  ┌──────────────┐
    │ PRIMITIVES │  │   SPECIAL  │  │   CUSTOM     │
    └────────────┘  └────────────┘  └──────────────┘
        │                │                │
    ┌───┼───────┐   ┌────┼────┐    ┌──────┼──────┐
    │   │       │   │    │    │    │      │      │
  string number  boolean any unknown void never  interface type class
  │     │       │               enum  union tuple
  └──────────────────────────────────────────────────

TYPE NARROWING FLOW
═══════════════════════════════════════════════════════════

Function receives value: unknown | string | number

           ↓
    ┌──────────────┐
    │ Type Guards  │
    └──────────────┘
           │
    ┌──────┴──────┐
    │             │
 typeof      instanceof
  check       check
    │             │
    ├─────┬───────┤
    │     │       │
  string number boolean
```

## Example

```typescript
// Basic types
const name: string = "Alice";
const age: number = 30;

// Union types
type Status = 'pending' | 'success' | 'error';
function handleStatus(status: Status): void {
  console.log(`Status: ${status}`);
}

// Type narrowing with type guard
function processId(id: string | number): string {
  if (typeof id === 'string') {
    return id.toUpperCase();
  }
  return id.toString();
}
```

## Usage

### When to Use This Knowledge

- **Type safety**: Catch errors at compile-time before runtime
- **IDE support**: Enable autocomplete and better IntelliSense
- **Documentation**: Types serve as self-documenting code
- **Refactoring**: Types help ensure changes don't break dependencies
- **Team collaboration**: Shared type definitions improve code clarity

### Real-world Example

```typescript
// API response typing
interface ApiResponse<T> {
  success: boolean;
  data?: T;
  error?: string;
}

interface Post {
  id: number;
  title: string;
  content: string;
  author: string;
}

// Fetch with proper typing
async function fetchPost(id: number): Promise<ApiResponse<Post>> {
  try {
    const response = await fetch(`/api/posts/${id}`);
    const data = await response.json();

    if (!response.ok) {
      return {
        success: false,
        error: data.message
      };
    }

    return {
      success: true,
      data: data as Post
    };
  } catch (error) {
    return {
      success: false,
      error: error instanceof Error ? error.message : 'Unknown error'
    };
  }
}

// Component props typing
interface ButtonProps {
  label: string;
  variant?: 'primary' | 'secondary' | 'danger';
  onClick: (event: React.MouseEvent<HTMLButtonElement>) => void;
  disabled?: boolean;
}

function Button({ label, variant = 'primary', onClick, disabled }: ButtonProps) {
  return (
    <button
      className={`btn btn-${variant}`}
      onClick={onClick}
      disabled={disabled}
    >
      {label}
    </button>
  );
}
```

### Best Practices

- **Use type inference** when types are obvious; don't over-annotate
- **Prefer interfaces** for object shapes; use types for unions and complex definitions
- **Avoid `any`** at all costs; use `unknown` and type guards instead
- **Use strict mode** (`"strict": true` in tsconfig.json)
- **Create reusable types** in separate files for consistency
- **Use readonly** for immutable properties to prevent mutations
- **Leverage discriminated unions** for complex type patterns
- **Document complex types** with JSDoc comments

## FAQ / Interview Questions

**Q: What's the difference between `any` and `unknown`?**
A: Both allow any value, but `unknown` is type-safe. With `any`, TypeScript trusts you and won't check anything. With `unknown`, you must narrow the type before using it. Use `unknown` as the default; only use `any` when you truly need to opt out of type checking.

**Q: How do type guards narrow union types?**
A: Type guards are conditional checks that help TypeScript understand a value's type within a specific code block. Using `typeof`, `instanceof`, or custom type guard functions, you narrow the union type. For example, in `if (typeof x === 'string')`, TypeScript knows `x` is a string inside the if-block, even though it started as `string | number`.

**Q: What's the difference between an interface and a type alias?**
A: Interfaces define object shapes and can be extended/merged. Types are more flexible and can represent unions, tuples, primitives, and other complex shapes. For most object definitions, interfaces are clearer; for complex types, use type aliases. Interfaces are also slightly better for performance and error messages.

**Q: What are generics and why are they useful?**
A: Generics are placeholders for types that are specified when the generic is used. They allow creating reusable components that work with different types while maintaining type safety. For example, `Array<T>` works with any type T (Array<string>, Array<number>, etc.). Generics prevent code duplication and provide flexibility without sacrificing type safety.

**Q: How do you handle functions that can accept multiple overloads?**
A: Use function overloads by declaring multiple function signatures followed by a single implementation. For example:
```typescript
function process(value: string): string;
function process(value: number): number;
function process(value: string | number): string | number {
  return typeof value === 'string' ? value.toUpperCase() : value * 2;
}
```
This tells TypeScript exactly what types are valid for each scenario.

## References

- [TypeScript Handbook - Basic Types](https://www.typescriptlang.org/docs/handbook/basic-types.html)
- [TypeScript Handbook - Advanced Types](https://www.typescriptlang.org/docs/handbook/advanced-types.html)
- [MDN - TypeScript](https://developer.mozilla.org/en-US/docs/Web/JavaScript/TypeScript)
- [TypeScript Deep Dive](https://basarat.gitbook.io/typescript/)

---

*See also: [Interface](./Interface.md), [Type](./Type.md), [As Const](./AsConst.md)*
