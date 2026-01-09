# Types in JavaScript

## Definition / Concept

**Types** in JavaScript represent the kind of data a value can hold. JavaScript is a **dynamically-typed** language, meaning types are determined at runtime and can change. There are two main categories: **primitive types** (immutable values stored directly) and **objects** (reference types). The language has 8 types: `undefined`, `null`, `boolean`, `number`, `string`, `symbol`, `bigint`, and `object` (which includes arrays, functions, and regular objects).

- **Primitive types** are immutable and stored by value
- **Object type** is mutable and stored by reference
- **Dynamic typing** allows values to change types at runtime
- **typeof operator** identifies type of a value
- **Type coercion** automatically converts between types

## Visual Representation

```
JAVASCRIPT TYPE HIERARCHY
═══════════════════════════════════════════════════════════

┌─────────────────────────────────────────────────────────┐
│                   All JavaScript Values                 │
└─────────────────────────────────────────────────────────┘
                            │
                ┌───────────┴───────────┐
                │                       │
        ┌──────────────┐        ┌───────────────┐
        │  PRIMITIVES  │        │   OBJECTS     │
        │ (By Value)   │        │ (By Reference)│
        └──────────────┘        └───────────────┘
              │                         │
    ┌─────────┼─────────────┐    ┌──────┼──────────┐
    │         │             │    │      │          │
undefined  null  boolean  number string Array  Function Object
                 │         │              │
               symbol   bigint        Regular Expr,
                                       Date, Map, etc.

STORAGE MODEL
═══════════════════════════════════════════════════════════

Primitives (Stored by Value):     Objects (Stored by Reference):
┌─────────────┐                   ┌────────────────────┐
│ let x = 5   │ stores 5          │ let obj = {}       │
│ let y = x   │ copies value      │ let ref = obj      │
│ y = 10      │ x still = 5       │ ref.name = "John"  │
│             │                   │ obj.name = "John"  │
└─────────────┘                   │ (both point same)  │
                                  └────────────────────┘
```

## Example

### Primitive Types

```javascript
// Strings - text data
const name = "Alice";
const message = 'Hello, World!';
const template = `Welcome, ${name}`;

// Numbers - integer and floating point
const age = 25;
const price = 19.99;
const infinity = Infinity;
const notANumber = NaN; // typeof "number"

// Boolean - true or false
const isActive = true;
const isValid = false;

// BigInt - large integers beyond Number limit
const largeNumber = 9007199254740992n; // n suffix indicates BigInt
const bigValue = BigInt("123456789012345678901234567890");

// Symbol - unique identifier
const id = Symbol('id');
const id2 = Symbol('id');
console.log(id === id2); // false - each symbol is unique

// undefined - variable declared but not assigned
let x;
console.log(x); // undefined

// null - intentional absence of value
const empty = null;
```

### Object Type (Reference Types)

```javascript
// Objects - collections of key-value pairs
const user = {
  name: "John",
  age: 30,
  isActive: true
};

// Arrays - ordered list of values
const colors = ["red", "green", "blue"];
const mixed = [1, "hello", true, null, { key: "value" }];

// Functions - reusable blocks of code
function greet(name) {
  return `Hello, ${name}`;
}
const arrow = (x) => x * 2;

// Built-in Objects
const date = new Date();
const regex = /[a-z]+/g;
const map = new Map();
const set = new Set([1, 2, 3]);
```

### Type Checking

```javascript
// typeof operator
typeof "hello";           // "string"
typeof 42;                // "number"
typeof true;              // "boolean"
typeof undefined;         // "undefined"
typeof null;              // "object" (famous quirk!)
typeof Symbol('id');      // "symbol"
typeof 42n;               // "bigint"
typeof {};                // "object"
typeof [];                // "object" (arrays are objects)
typeof function(){};      // "function"

// instanceof for objects
[] instanceof Array;      // true
/test/ instanceof RegExp; // true

// Object.prototype.toString for precise type
Object.prototype.toString.call([]);     // "[object Array]"
Object.prototype.toString.call({});     // "[object Object]"
Object.prototype.toString.call(null);   // "[object Null]"
```

### Type Coercion

```javascript
// Implicit coercion (automatic)
"5" + 3;              // "53" (concatenation)
5 + "3";              // "53" (number coerced to string)
"10" - 3;             // 7 (string coerced to number)
true + 1;             // 2 (boolean coerced to number)
null + 5;             // 5 (null coerced to 0)
undefined + 5;        // NaN (undefined coerced to NaN)

// Explicit coercion (intentional)
String(42);           // "42"
Number("42");         // 42
Boolean(0);           // false
Boolean("hello");     // true

// Truthy and Falsy values
if ("") {} // false (empty string)
if (0) {}  // false
if (-0) {} // false
if (NaN) {} // false
if (null) {} // false
if (undefined) {} // false
if (false) {} // false
if (true) {} // true
if ("hello") {} // true
if (1) {} // true
if ([]) {} // true (even empty array!)
if ({}) {} // true (even empty object!)
```

## Usage

### When to Use This Knowledge

- **Type checking**: Validate data types before performing operations
- **Preventing bugs**: Understand coercion to avoid unexpected behavior
- **API design**: Enforce expected types in function parameters
- **Debugging**: Identify type-related errors quickly
- **Performance**: Choose appropriate types for memory efficiency

### Real-world Example

```javascript
// User input validation
function processUserInput(input) {
  // Always check types when accepting user data
  if (typeof input !== 'string') {
    throw new Error('Input must be a string');
  }

  const trimmed = input.trim();
  return trimmed.length > 0 ? trimmed : null;
}

// API response handling
async function fetchUser(id) {
  const response = await fetch(`/api/users/${id}`);
  const data = await response.json();

  // Validate returned types
  if (typeof data.id !== 'number') {
    throw new Error('Invalid user ID type');
  }

  return {
    id: data.id,
    name: String(data.name || 'Unknown'),
    isActive: Boolean(data.isActive),
    roles: Array.isArray(data.roles) ? data.roles : []
  };
}

// Type guards in functions
function processValue(value) {
  if (typeof value === 'number') {
    return value * 2;
  } else if (typeof value === 'string') {
    return value.toUpperCase();
  } else if (Array.isArray(value)) {
    return value.length;
  } else {
    return null;
  }
}
```

### Best Practices

- **Use strict equality** (`===`) to avoid type coercion surprises
- **Declare types early** with consistent naming (e.g., `isValid` for booleans)
- **Validate external data** (user input, API responses) at system boundaries
- **Avoid implicit coercion** in critical code; use explicit conversion
- **Use Array.isArray()** instead of `typeof` for array checking
- **Consider TypeScript** for larger projects requiring type safety

## FAQ / Interview Questions

**Q: What are the primitive types in JavaScript?**
A: JavaScript has 8 primitive types: `undefined`, `null`, `boolean`, `number`, `string`, `symbol`, `bigint`, and `object`. Primitives are immutable and stored by value. The `object` type technically includes arrays, functions, and other built-in objects, though they're reference types.

**Q: Why does `typeof null` return "object"?**
A: This is a famous quirk in JavaScript's design. It's actually a bug that became standardized. In the original JavaScript implementation, `null` was represented as a null pointer (0x00), which matched the object type tag, so `typeof` incorrectly reports it as "object". To check for null, use `value === null` instead of `typeof`.

**Q: What's the difference between `undefined` and `null`?**
A: `undefined` is the default value for variables that haven't been assigned anything and is returned by functions without a return statement. `null` is an intentional representation of "no value" that must be explicitly assigned. Both are falsy, but `undefined == null` is true while `undefined === null` is false.

**Q: Explain type coercion. Why is `"5" + 3 = "53"` but `"5" - 3 = 2`?**
A: Type coercion is JavaScript's automatic conversion between types. The `+` operator is special—it first tries to convert both operands to strings for concatenation, so `"5" + 3` becomes `"53"`. The `-` operator only works with numbers, so JavaScript converts both operands to numbers: `"5"` becomes `5`, and `5 - 3 = 2`. This is why understanding operator behavior is crucial.

**Q: What's the difference between checking `typeof x === "undefined"` and `x === undefined`?**
A: Both work, but `typeof x === "undefined"` is safer because it won't throw an error if `x` is not declared at all (due to how `typeof` works with undeclared variables). Using `x === undefined` will throw a ReferenceError if `x` is completely undeclared. However, for declared but uninitialized variables, both work identically.

## References

- [MDN - JavaScript Data Types](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Data_structures)
- [MDN - typeof Operator](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/typeof)
- [MDN - Equality Comparisons](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Equality_comparisons_and_sameness)
- [JavaScript.info - Data Types](https://javascript.info/types)

---

*See also: [Typeof](./Typeof.md), [Equality](./Equality.md), [Scope Chain](./ScopeChain.md), [This](./This.md)*
