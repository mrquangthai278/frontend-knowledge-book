# Hoisting

## Definition / Concept

**Hoisting** is JavaScript's behavior of moving declarations to the top of their scope before code execution. This happens during the compilation phase, not at runtime. Variables declared with `var` are hoisted with the value `undefined`, while `let` and `const` are hoisted but not initialized, creating a **Temporal Dead Zone (TDZ)** — a period where accessing the variable throws a `ReferenceError`.

- **`var` declarations** are hoisted and initialized with `undefined`
- **`let` and `const` declarations** are hoisted but not initialized
- **Function declarations** are fully hoisted and can be called before declaration
- **Function expressions** behave like variable declarations

## Visual Representation

```
CODE EXECUTION TIMELINE
═══════════════════════════════════════════════════════════

Before Hoisting:
┌─────────────────────────────────────┐
│ console.log(x);  // ReferenceError  │
│ let x = 5;                          │
│ console.log(x);  // 5               │
└─────────────────────────────────────┘

After Hoisting (Compilation Phase):
┌─────────────────────────────────────┐
│ let x;           // Hoisted (TDZ)   │
│ console.log(x);  // ReferenceError  │
│ x = 5;           // Assignment      │
│ console.log(x);  // 5               │
└─────────────────────────────────────┘

Temporal Dead Zone (TDZ):
┌──────────────────────────────────┐
│ START (scope created)            │
│ ↓                                │
│ [TDZ - can't access x]           │
│ ↓                                │
│ let x;  ← Declaration reached    │
│ ↓                                │
│ x = 5;  ← Now x can be used      │
└──────────────────────────────────┘
```

## Example

### var - Hoisted and Initialized

```javascript
console.log(name);  // undefined (hoisted + initialized)
var name = "John";
// Interpreted as: var name; [hoisted] then console.log, then assignment
```

### let/const - Hoisted but Not Initialized (TDZ)

```javascript
console.log(age);   // ReferenceError: Cannot access before initialization
let age = 25;       // TDZ ends here at declaration
```

### Function Declaration vs Expression

```javascript
sayHello();         // Works fine! (fully hoisted)
function sayHello() { console.log("Hello!"); }

greet();            // TypeError: not a function (only var is hoisted)
var greet = function() { console.log("Hi!"); };
```

### Loop Variable Hoisting

```javascript
// Bad: var (all iterations share same i)
for (var i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);  // 3, 3, 3
}

// Good: let (each iteration has its own i)
for (let i = 0; i < 3; i++) {
  setTimeout(() => console.log(i), 0);  // 0, 1, 2
}
```

## Usage

### When to Use This Knowledge

- **Avoid hoisting bugs**: Always declare variables before using them
- **Choose the right keyword**: Use `let`/`const` instead of `var` to avoid hoisting confusion
- **Understand TDZ**: Know that `let` and `const` variables are not accessible before their declaration
- **Debug scope issues**: Understanding hoisting helps identify scope-related bugs
- **Interview preparation**: Hoisting is a common interview question about JavaScript fundamentals

### Real-world Example

```javascript
// Without understanding hoisting, this confuses beginners
var config = {};
function init() {
  console.log(config);  // undefined (hoisted but not initialized)
  config = { port: 3000 };
}

// Better: use let/const to avoid hoisting confusion
const PORT = process.env.PORT || 3000;
```

### Best Practices

- **Always use `let` or `const`** instead of `var` (modern JavaScript)
- **Declare variables at the top** of their scope for clarity
- **Initialize variables** before using them to avoid Temporal Dead Zone
- **Use `const` by default**, `let` when you need to reassign
- **Understand block scope** with `let` and `const` to prevent hoisting issues

## FAQ / Interview Questions

**Q: What is hoisting and how does it work?**
A: Hoisting is JavaScript's behavior where variable and function declarations are moved to the top of their scope during compilation. For `var`, the variable is initialized as `undefined`. For `let` and `const`, declarations are hoisted but not initialized, creating a Temporal Dead Zone where accessing them throws an error.

**Q: What's the difference between hoisting `var`, `let`, and `const`?**
A: `var` is hoisted and initialized with `undefined`, allowing you to access it before declaration. `let` and `const` are hoisted but not initialized, causing a ReferenceError if accessed before declaration (Temporal Dead Zone). All three are hoisted; the difference is in initialization.

**Q: Why does this output undefined instead of throwing an error?**
```javascript
console.log(x);  // undefined
var x = 5;
```
A: Because `var` is hoisted and initialized with `undefined`. The code is rewritten as:
```javascript
var x;           // Declaration + initialization to undefined
console.log(x);  // undefined
x = 5;           // Assignment
```

**Q: What is the Temporal Dead Zone (TDZ)?**
A: The Temporal Dead Zone is the period from when a `let` or `const` variable is hoisted until its declaration statement is executed. Accessing the variable during TDZ throws a `ReferenceError`. It creates safer code by forcing you to declare variables before use.

**Q: Why should we avoid using `var`?**
A: `var` has confusing hoisting behavior and function scope instead of block scope, leading to bugs. `let` and `const` are block-scoped and have the Temporal Dead Zone, making code more predictable and safer. Using `let`/`const` prevents accidental re-declarations and encourages better coding practices.

## References

- [MDN - Hoisting](https://developer.mozilla.org/en-US/docs/Glossary/Hoisting)
- [MDN - Temporal Dead Zone](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let#Temporal_Dead_Zone)
- [JavaScript.info - Variable Scope, Closure](https://javascript.info/closure)
- [ES6 Standard - Let and Const](https://www.ecma-international.org/ecma-262/)

---

*See also: [Scope](./Scope.md), [Closure](./Closure.md), [Typeof](./Typeof.md), [Temporal Dead Zone](./TemporalDeadZone.md)*
