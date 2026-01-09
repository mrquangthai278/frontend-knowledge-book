# as const

## Definition / Concept

**`as const`** is a TypeScript assertion that tells the compiler to infer the most specific literal type for a value, rather than generalizing it to a broader type. Instead of treating `"hello"` as type `string`, it treats it as the literal type `"hello"`. This is especially powerful for creating **readonly tuples** and **enum-like objects** while maintaining strict type safety.

- Creates **literal types** instead of wider primitive types
- Makes objects and arrays **readonly** at the type level
- Useful for creating **type-safe constants** and configuration objects
- Enables **discriminated unions** and **exhaustiveness checking**

## Visual Representation

```
Without as const:
const status = "success"  → type: string (very broad)

With as const:
const status = "success" as const  → type: "success" (very specific)

Object without as const:
const colors = { red: "#FF0000" }  → { red: string }

Object with as const:
const colors = { red: "#FF0000" } as const  → { readonly red: "#FF0000" }
```

## Example

```typescript
// Without as const - types are too broad
const status = "success";  // type: string
const config = { port: 3000, host: "localhost" };  // { port: number; host: string; }

// With as const - types are specific and readonly
const statusConst = "success" as const;  // type: "success"
const configConst = { port: 3000, host: "localhost" } as const;  // { readonly port: 3000; readonly host: "localhost"; }

// Creating type-safe enums
const COLORS = {
  RED: "#FF0000",
  GREEN: "#00FF00",
  BLUE: "#0000FF"
} as const;

type ColorType = typeof COLORS[keyof typeof COLORS];  // "#FF0000" | "#00FF00" | "#0000FF"

// Using with discriminated unions
type Success = { status: "success"; data: unknown } as const;
type Error = { status: "error"; message: string } as const;
type Result = Success | Error;

const result: Result = { status: "success", data: { id: 1 } };

// Tuples with as const
const coordinates = [10, 20] as const;  // type: readonly [10, 20]
const [x, y] = coordinates;  // x: 10, y: 20 (literal types!)
```

## Usage

- **When to use**: Creating constants, configuration objects, enums, and discriminated union types where you need strict literal types
- **Real-world example**: API response handlers with status codes, theme configuration objects, feature flags
- **Best practices**:
  - Use `as const` for constants that shouldn't change
  - Prefer `as const` objects over enums for simple mappings
  - Combine with `typeof` and `keyof` for powerful type extraction
  - Use at the end of declarations (e.g., `const x = {...} as const`)

## FAQ / Interview Questions

**Q: What's the difference between `as const` and regular const?**
A: Both `const` and `as const` prevent reassignment. The difference is in **type inference**: `const x = "hello"` has type `string`, while `const x = "hello" as const` has type `"hello"` (the literal). Regular `const` just prevents reassignment; `as const` also narrows the type to its most specific form.

**Q: When would you use `as const` instead of an enum?**
A: `as const` is lighter weight and more flexible than enums. Use `as const` for simple mappings, configuration objects, or when you need to export values that work well with `typeof`. Use enums when you need more features like reverse mappings or when enum semantics are important to your codebase.

**Q: Can you use `as const` with arrays?**
A: Yes! `as const` on arrays creates readonly tuples with literal types. For example, `[1, 2, 3] as const` becomes `readonly [1, 2, 3]` instead of `number[]`. This is useful for fixed-size tuples where each element has a specific type.

**Q: How do you extract types from a `const` object?**
A: Use `typeof` and `keyof` together:
```typescript
const config = { host: "localhost", port: 3000 } as const;
type Config = typeof config;  // { readonly host: "localhost"; readonly port: 3000; }
type Keys = keyof typeof config;  // "host" | "port"
type Values = typeof config[keyof typeof config];  // "localhost" | 3000
```

**Q: Is `as const` the same as readonly?**
A: Similar but not identical. `as const` makes properties readonly AND infers their literal types. Just using `readonly` only prevents modification but doesn't narrow types. `as const` is more powerful for type inference.

## References
- [TypeScript Handbook - Literal Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#literal-types)
- [TypeScript Handbook - const Assertions](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-4.html#const-assertions)
- [TypeScript Deep Dive - Type Inference and Widening](https://basarat.gitbook.io/typescript/type-system/type-inference#widening)
- [MDN - JavaScript const Statement](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/const)

---

*See also: [TypeScript Literals](./Literals.md), [Enums](./Enums.md), [typeof](./Typeof.md), [Type Extraction](./TypeExtraction.md)*
