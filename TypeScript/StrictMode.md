# Strict Mode

## Definition / Concept

**TypeScript Strict Mode** is a compiler option that enables a set of strict type-checking rules to ensure higher code quality and type safety. When enabled with `"strict": true` in `tsconfig.json`, it activates multiple sub-options that catch potential errors during development rather than at runtime. This mode enforces stricter interpretations of the type system, preventing common mistakes like accessing properties on potentially null values, implicit `any` types, and incomplete property assignments.

- Catches type errors early in development
- Prevents implicit `any` type declarations
- Enforces null and undefined type safety
- Ensures better code maintainability and reliability

## Visual Representation

```
Strict Mode Hierarchy
├── noImplicitAny (forbid implicit any types)
├── noImplicitThis (forbid implicit this: any)
├── strictNullChecks (null/undefined not assignable to other types)
├── strictFunctionTypes (stricter function type checking)
├── strictBindCallApply (stricter bind, call, apply)
├── strictPropertyInitialization (non-optional properties must be initialized)
└── noImplicitReturns (all code paths must return a value)
```

## Example

```typescript
// Without Strict Mode - potential errors not caught
function greet(name) {  // implicit any
  return "Hello " + name;
}

let user = { name: "John" };
console.log(user.age);  // undefined, no error

let value: string = null;  // assignable without strict mode

// With Strict Mode enabled - errors caught
// ❌ Parameter 'name' implicitly has an 'any' type
function greet(name: string) {
  return "Hello " + name;
}

// ❌ Property 'age' does not exist on type '{ name: string }'
let user = { name: "John" };
console.log(user.age);

// ❌ Type 'null' is not assignable to type 'string'
let value: string = null;

// ✅ Correct approach with strict mode
function greet(name: string): string {
  return "Hello " + name;
}

let user: { name: string } = { name: "John" };
console.log(user.name);

let value: string | null = null;  // explicit optional
let message: string = value ?? "default";  // safe handling
```

## Usage

- **When to use**: Always enable strict mode for new projects to prevent bugs and maintain code quality
- **Real-world example**: In a React component, strict mode catches when you try to access properties that might be undefined:
  ```typescript
  interface User {
    name: string;
    email: string;
  }

  function UserCard({ user }: { user: User | null }) {
    // ❌ Error: user might be null
    return <h1>{user.name}</h1>;

    // ✅ Correct: check for null first
    return user ? <h1>{user.name}</h1> : <p>No user</p>;
  }
  ```
- **Best practices**:
  - Enable `"strict": true` in `tsconfig.json` for all new projects
  - Gradually enable strict mode in existing projects by enabling sub-options individually
  - Use type guards and optional chaining (`?.`) to safely access properties
  - Leverage the `??` (nullish coalescing) operator for default values

## FAQ / Interview Questions

**Q: What is the difference between strict mode and `noImplicitAny`?**
A: Strict mode is a collection of type-checking rules that includes `noImplicitAny` plus six other options. `noImplicitAny` specifically prevents variables and parameters from having implicit `any` types, while strict mode provides comprehensive type safety covering null checks, function types, property initialization, and more.

**Q: How do I enable strict mode in TypeScript?**
A: Add `"strict": true` to your `compilerOptions` in `tsconfig.json`. This automatically enables all strict sub-options. You can also enable individual options like `"noImplicitAny": true` or `"strictNullChecks": true` without enabling the full strict mode.

**Q: Why does TypeScript throw an error when I assign `null` to a string type with strict mode?**
A: With `strictNullChecks` enabled (part of strict mode), `null` and `undefined` are not considered valid values for any type. This prevents null reference errors at runtime. To allow null values, use a union type: `let value: string | null = null;`

**Q: Can I enable strict mode gradually in an existing project?**
A: Yes, you can enable strict sub-options individually in `tsconfig.json` without enabling full strict mode. Start with options like `noImplicitAny` and gradually enable others. Alternatively, enable strict mode and then set specific sub-options to `false` to exclude them temporarily.

**Q: What is the `strictPropertyInitialization` rule?**
A: This rule requires all class properties to be initialized in the constructor or with a default value. Without it, TypeScript doesn't check if properties declared as non-optional are actually assigned, which can lead to undefined values at runtime.

## References

- [TypeScript Handbook - strict](https://www.typescriptlang.org/tsconfig#strict)
- [TypeScript Handbook - Strict Mode Checks](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#the-typescript-reference)
- [TypeScript Official Documentation - Type Checking](https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html)

---

*See also: [Types](./Types.md), [Interface](./Interface.md), [Type](./Type.md)*
